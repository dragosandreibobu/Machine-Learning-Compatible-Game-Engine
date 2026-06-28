# Editor CLI and Process Boundary Debt

## The uncomfortable truth

The current Python editor prototype uses command-line subprocess calls as an internal communication mechanism. That made sense for fast thesis/demo progress, but it is not a good long-term architecture.

The pattern looks roughly like this:

```text
GUI widget
    -> subprocess: python -m Some.Editor.Module --some-arg value
        -> module parses argv
            -> module prints a tagged line
                -> GUI parses stdout
```

This creates a fragile system where editor state is passed through process arguments, stdout strings, and implicit conventions instead of through typed in-memory APIs.

## Why it became spaghetti

This style usually starts harmlessly:

```text
"I just need the GUI to ask the file manager whether this path is valid."
```

Then it spreads:

```text
open project
create project
get latest scene
get config
create empty scene
run project
```

Every interaction becomes a mini command-line protocol. Eventually the editor is not one application anymore; it is a network of small programs calling each other by arguments and parsing each other's printed output.

That is workable for a demo. It is painful for an engine editor.

## Problems caused by internal CLI calls

### 1. No typed boundary

A function call can return a `Project`, `Scene`, `Path`, or `Result<T>`.

A subprocess returns:

```text
stdout
stderr
exit code
```

The caller then has to search for magic prefixes like:

```text
isValidProject: true
LatestSavedScene: path/to/file.json
```

That means the real API is hidden in string formatting.

### 2. Weak error handling

When the subprocess fails, the caller often knows only that the command failed. It may not know whether the issue was:

- missing project file;
- malformed JSON;
- missing scene folder;
- invalid command arguments;
- Python import failure;
- an internal exception;
- a user-facing validation error.

A real API can return structured errors. A subprocess protocol usually degrades into printing text.

### 3. State fragmentation

The editor should have a shared in-memory concept of:

```text
current project
active scene
selected object
dirty state
asset root
play/edit mode
```

With subprocess calls, each command reconstructs its own partial view from paths and arguments. State becomes scattered instead of owned by an `EditorContext`.

### 4. Harder refactoring

If a GUI panel depends on a printed line from a helper command, renaming that line becomes a breaking API change.

The protocol is not discoverable through the type system. It is only discoverable by reading both the caller and the subprocess implementation.

### 5. Bad fit for a C++/Qt rewrite

A future C++/Qt editor should not preserve this style. The rewrite should replace internal process calls with direct services:

```cpp
ProjectService projectService;
SceneService sceneService;
AssetService assetService;
BuildService buildService;
```

The editor should call these services directly and receive typed results.

## What the future boundary should look like

The editor should have a central context:

```cpp
class EditorContext
{
public:
    Project project;
    Scene activeScene;
    GameObjectId selectedObject;
    bool isDirty = false;
    bool isPlaying = false;
};
```

Panels should communicate through this context and services:

```text
HierarchyPanel
    reads EditorContext.activeScene
    writes EditorContext.selectedObject

InspectorPanel
    reads selected GameObject
    mutates component data
    marks scene dirty

SceneViewport
    renders EditorContext.activeScene

AssetsPanel
    reads Project asset roots

ProjectService
    opens/saves project data

SceneService
    loads/saves scene data
```

## Recommended service layer

A clean C++/Qt editor could use services like:

```cpp
class ProjectService
{
public:
    Result<Project> openProject(const Path& path);
    Result<void> saveProject(const Project& project);
    Result<Project> createProject(const ProjectCreateInfo& info);
};

class SceneService
{
public:
    Result<Scene> loadScene(const Path& path);
    Result<void> saveScene(const Scene& scene, const Path& path);
    Scene createEmptyScene(std::string name);
};

class AssetService
{
public:
    std::vector<AssetInfo> listAssets(const Project& project);
    Result<AssetInfo> importAsset(const Path& source, const Path& destination);
};

class BuildService
{
public:
    Result<BuildArtifact> buildProject(const Project& project, BuildTarget target);
    Result<void> runProject(const BuildArtifact& artifact);
};
```

The GUI remains a GUI. The engine/editor logic lives in testable services.

## What should still be allowed to use processes

Not every subprocess is bad. Some boundaries are naturally process-based:

- launching a standalone game runtime;
- compiling a project;
- invoking an external compiler/toolchain;
- starting a web export pipeline;
- opening a file browser;
- running optional external ML/data tools.

The rule should be:

```text
Use processes for external tools and runtime boundaries.
Do not use processes for internal editor state and data flow.
```

## Migration path from the current prototype

### Step 1: Name the anti-pattern

Document that subprocess-based editor communication was prototype glue, not the intended architecture.

### Step 2: Extract pure functions

Move logic like project validation, project creation, scene loading, and scene discovery into pure functions or classes.

Bad:

```text
GUI -> subprocess -> argparse -> filesystem -> print result -> parse stdout
```

Better:

```text
GUI -> ProjectService::openProject(path) -> Result<Project>
```

### Step 3: Keep CLI wrappers only as thin shells

If command-line tools are still useful, make them wrappers around the real service layer:

```text
CLI command
    -> ProjectService
        -> typed result
            -> formatted terminal output
```

The CLI should not be the source of truth.

### Step 4: In the C++/Qt rewrite, start with services

Before building every panel, create:

```text
EditorContext
ProjectService
SceneService
AssetService
BuildService
```

Then panels can remain simple and avoid spaghetti from the beginning.

## Desired end state

```text
Editor UI
    -> typed services
        -> engine/editor data model
            -> serializers/build/runtime hosts
```

Not:

```text
Editor UI
    -> subprocess
        -> argparse
            -> print tagged stdout
                -> parse string in another widget
```

The current CLI-heavy approach was useful glue for getting the demo moving. The next architecture should treat it as technical debt to retire, not as a pattern to preserve.
