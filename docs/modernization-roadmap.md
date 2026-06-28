# Modernization Roadmap

This roadmap focuses on turning the prototype into a coherent next-generation engine/editor system without losing the simplicity and beauty of the original engine API.

## Phase 0: Preserve the prototype

Before rewriting, preserve what exists:

- keep current demos available;
- document current engine API;
- keep the Python editor as historical/prototype reference;
- avoid rewriting everything before proving the new architecture.

## Phase 0.5: Retire internal CLI glue

The Python prototype used subprocess calls and argument parsing for internal editor communication. Do not carry that pattern into the C++/Qt rewrite. Before building new panels, define typed services for project, scene, asset, and build operations. CLI commands may remain as external wrappers, but they should call the same services rather than being the primary editor API.

## Phase 1: CMake and C++ project structure

Create a modern C++ layout:

```text
Engine/
EditorQt/
Runtime/
Tests/
Projects/
```

Use CMake targets:

```text
bleuemer_engine
bleuemer_editor
bleuemer_runtime
bleuemer_tests
```

Goal: compile the engine as a reusable library.

## Phase 2: Loop-agnostic engine core

Extract the engine from hard ownership of any one windowing loop.

Target API:

```cpp
engine.initialize();
engine.update(scene, dt);
engine.render(scene, renderer);
engine.shutdown();
```

Success condition: a test or minimal host can call update/render without entering a GLUT-style permanent main loop.

## Phase 3: Scene and GameObject model

Add first-class runtime types:

```text
Scene
GameObject
Transform
RendererComponent
Rigidbody
Collider
Behaviour
```

The current fixed-component style can be preserved initially. The first goal is not a perfect ECS. The first goal is a stable, serializable runtime object model.

## Phase 4: Render command queue

Unify immediate drawing and GameObject rendering through commands:

```text
draw.circle(...)
    -> RenderCommand

object.renderer.render(...)
    -> RenderCommand

backend.flush(commands)
    -> OpenGL/WebGL draw calls
```

Success condition: Processing-style calls and scene-object rendering share the same backend path.

## Phase 5: Scene JSON serialization

Implement:

```text
ProjectSerializer
SceneSerializer
ComponentSerializer
```

Success condition:

```text
Scene JSON -> C++ Scene -> rendered objects
```

This is the central architecture proof.

## Phase 6: Minimal C++/Qt editor

Do not recreate every Python editor feature first. Build only:

- main window;
- hierarchy panel;
- inspector panel;
- `QOpenGLWidget` scene viewport;
- asset browser stub.

Success condition:

```text
Open scene JSON
    -> hierarchy displays objects
    -> inspector edits transform/color
    -> viewport updates
    -> save writes JSON
```

## Phase 7: Native runtime host

Create a standalone runtime executable:

```text
bleuemer_runtime Projects/dvd_bouncer/ProjectData.json
```

Success condition:

```text
runtime loads project
runtime loads active scene
runtime runs update/render loop
```

## Phase 8: Processing-style project layer

Add a clean sketch API:

```cpp
class Sketch
{
public:
    virtual void start() {}
    virtual void update(float dt) {}
};
```

Allow projects to combine:

```text
scene-loaded GameObjects
+ immediate Processing-style drawing
+ custom C++ behavior
```

## Phase 9: ML-native module

Implement a first ML slice:

1. Matrix/Tensor basics;
2. Dataset container;
3. LinearRegression;
4. model serialization;
5. `MLBehaviour` component;
6. ML-controlled paddle demo.

Success condition:

```text
GameObject behavior uses native C++ model prediction during update
```

## Phase 10: Web export

Compile engine/runtime to WASM after the loop-agnostic architecture is proven.

Export package:

```text
dist/
  index.html
  engine.wasm
  project.json
  assets/
    scenes/
    models/
```

Success condition:

```text
same scene JSON runs natively and in browser
```

## Recommended order of proof

The strongest proof sequence is:

```text
1. C++ engine library
2. Scene JSON loads into C++ objects
3. Objects render through command queue
4. Qt viewport renders the same scene
5. Runtime executable loads the same scene
6. ML component drives an object
7. WASM runtime loads the same scene
```

Do not start with web deployment or full ML tooling. Build the shared core first.
