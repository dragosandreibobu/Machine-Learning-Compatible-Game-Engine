# Branch and Pull Request Plan

This plan describes the order of future branches and pull requests for turning the prototype into a coherent engine/editor stack. The goal is to keep every PR reviewable, preserve the existing demos, and avoid mixing architecture, build-system work, editor rewrites, runtime changes, ML work, and web export in one giant branch.

## Branch naming convention

Use short branch names with an ordered prefix:

```text
roadmap/00-docs-and-inventory
engine/01-cmake-library
engine/02-loop-agnostic-core
engine/03-scene-model
engine/04-render-command-queue
serialization/05-scene-json
editor/06-qt-shell
editor/07-qt-scene-editing
runtime/08-native-player
sketch/09-processing-layer
ml/10-core-models
ml/11-ml-behaviour-demo
web/12-wasm-export
cleanup/13-retire-python-editor-glue
```

The number is not about urgency alone. It is about dependency order.

## PR 00: Documentation and repository inventory

**Branch:** `roadmap/00-docs-and-inventory`

**Purpose:** Establish shared language before code starts moving.

**Should include:**

- architecture documentation;
- current repository inventory;
- list of demos and their dependencies;
- known technical debt;
- decision log for C++/Qt, JSON scenes, loop ownership, ML-native direction, and web deployment.

**Should not include:**

- large code movement;
- build-system rewrite;
- editor rewrite.

**Exit criteria:**

- contributors understand the intended future shape;
- old prototype shortcuts are explicitly marked as prototype shortcuts;
- no one has to infer the architecture only from code.

## PR 01: CMake engine library foundation

**Branch:** `engine/01-cmake-library`

**Purpose:** Make the C++ engine build as a reusable library target.

**Should include:**

- root `CMakeLists.txt`;
- `bleuemer_engine` target;
- existing engine sources compiled through CMake;
- minimal example/demo target if practical;
- documented build commands.

**Should not include:**

- Qt editor;
- scene serialization;
- renderer redesign;
- ML.

**Exit criteria:**

```text
cmake -S . -B build
cmake --build build
```

builds the engine target.

## PR 02: Loop-agnostic engine core

**Branch:** `engine/02-loop-agnostic-core`

**Purpose:** Separate engine update/render calls from GLUT owning the entire lifecycle.

**Should include:**

- an `Engine` or `Application` core API;
- `initialize`, `update`, `render`, and `shutdown` style functions;
- legacy GLUT host adapted to call the new API;
- no behavior change for existing demos if possible.

**Should not include:**

- full renderer abstraction;
- Qt viewport;
- scene JSON;
- web export.

**Exit criteria:**

- the engine can be advanced one frame by a host;
- GLUT becomes one host, not the engine architecture itself;
- future Qt and web hosts have a clean entry point.

## PR 03: Runtime scene and GameObject model

**Branch:** `engine/03-scene-model`

**Purpose:** Add first-class `Scene` ownership around the existing `GameObject` direction.

**Should include:**

- `Scene` type;
- object collection;
- scene-level update/render traversal;
- stable IDs or handles if needed;
- a minimal behavior interface if it can remain small.

**Should not include:**

- JSON loading yet;
- full ECS;
- Qt inspector;
- ML behavior.

**Exit criteria:**

- code can create a `Scene`, add objects, update it, and render it;
- existing immediate drawing style still works.

## PR 04: Render command queue

**Branch:** `engine/04-render-command-queue`

**Purpose:** Make Processing-style drawing and `GameObject` rendering share one rendering path.

**Should include:**

- `RenderCommand` type;
- command buffer or queue;
- primitive draw commands for rect, circle, line, point;
- backend flush path for the existing OpenGL implementation.

**Should not include:**

- Qt renderer backend;
- WebGL backend;
- material/mesh system unless absolutely minimal.

**Exit criteria:**

- immediate draw calls produce commands;
- object rendering produces commands;
- existing OpenGL backend can draw those commands.

## PR 05: Scene JSON serialization

**Branch:** `serialization/05-scene-json`

**Purpose:** Make scenes loadable and saveable as data.

**Should include:**

- project metadata shape;
- scene JSON parser/writer;
- component serialization for transform, renderer, rigidbody, collider;
- sample scene file;
- tests for load/save roundtrips if possible.

**Should not include:**

- Qt editor;
- web export;
- ML model serialization.

**Exit criteria:**

```text
Scene JSON -> C++ Scene -> renderable objects
C++ Scene -> Scene JSON
```

works for the basic object model.

## PR 06: Minimal C++/Qt editor shell

**Branch:** `editor/06-qt-shell`

**Purpose:** Create the new editor application without recreating every old panel.

**Should include:**

- Qt application target;
- main window;
- dock/panel layout;
- empty hierarchy, inspector, assets, console, and scene viewport placeholders;
- shared `EditorContext` type.

**Should not include:**

- full scene editing;
- ML panel;
- web export;
- replacing old Python editor yet.

**Exit criteria:**

- C++/Qt editor opens;
- panels are visible;
- it links against the engine library.

## PR 07: Qt scene loading, hierarchy, inspector, and viewport

**Branch:** `editor/07-qt-scene-editing`

**Purpose:** Make the Qt editor operate on real engine scene data.

**Should include:**

- open project/scene flow;
- hierarchy populated from `Scene`;
- object selection;
- inspector editing for transform and renderer color/shape;
- `QOpenGLWidget` scene rendering;
- save scene.

**Should not include:**

- full asset importing;
- advanced gizmos;
- ML panel;
- web export.

**Exit criteria:**

```text
Open scene JSON
    -> hierarchy shows objects
    -> inspector edits object
    -> viewport updates
    -> save writes JSON
```

## PR 08: Native runtime/player host

**Branch:** `runtime/08-native-player`

**Purpose:** Create a standalone player that loads and runs project data outside the editor.

**Should include:**

- runtime executable target;
- project/scene loading;
- native window host;
- input forwarding;
- update/render loop using the loop-agnostic engine API.

**Should not include:**

- editor features;
- web export;
- ML training UI.

**Exit criteria:**

```text
bleuemer_runtime path/to/ProjectData.json
```

loads the active scene and runs it.

## PR 09: Processing-style sketch layer

**Branch:** `sketch/09-processing-layer`

**Purpose:** Preserve and formalize the Processing-like authoring style.

**Should include:**

- `Sketch` or similar base class/interface;
- `start/setup` and `update/draw` style lifecycle;
- immediate drawing wrappers over the render command queue;
- example sketch project.

**Should not include:**

- editor rewrite work;
- ML;
- web export.

**Exit criteria:**

- a code-first Processing-style project can run;
- a hybrid project can combine scene-loaded objects and immediate drawing.

## PR 10: Native ML core primitives

**Branch:** `ml/10-core-models`

**Purpose:** Add the first small C++ ML module.

**Should include:**

- small matrix/vector helpers if needed;
- dataset container;
- `Model` interface;
- `LinearRegression` as the first implemented model;
- simple tests or example usage.

**Should not include:**

- editor ML panel;
- every scikit-learn algorithm;
- object behavior integration yet.

**Exit criteria:**

- a model can be fit or loaded;
- prediction works in C++ without Python runtime dependency.

## PR 11: ML behaviour and demo integration

**Branch:** `ml/11-ml-behaviour-demo`

**Purpose:** Connect ML to the engine object model.

**Should include:**

- `MLBehaviour` or equivalent component/behavior;
- model input/output binding concept;
- first demo such as ML-controlled paddle;
- model serialization if not already included.

**Should not include:**

- full ML editor tooling;
- complex neural networks;
- web export unless trivial.

**Exit criteria:**

- a `GameObject` can use native C++ model prediction during update.

## PR 12: Web/WASM export proof

**Branch:** `web/12-wasm-export`

**Purpose:** Prove the architecture can run in a browser.

**Should include:**

- Emscripten build path;
- minimal web host;
- asset loading strategy;
- canvas/WebGL rendering path or compatible backend;
- sample exported scene.

**Should not include:**

- complete web editor;
- advanced asset pipeline;
- large ML demos.

**Exit criteria:**

- the same simple scene can run natively and in a browser.

## PR 13: Retire or quarantine Python editor glue

**Branch:** `cleanup/13-retire-python-editor-glue`

**Purpose:** Reduce confusion once the C++/Qt path exists.

**Should include:**

- mark old Python editor as legacy, historical, or prototype;
- move it under a clear legacy folder if desired;
- remove internal CLI glue from the active path;
- keep samples if they are still useful.

**Should not include:**

- risky deletion before replacement exists;
- changing engine behavior.

**Exit criteria:**

- the active architecture is C++ engine + C++/Qt editor + runtime host;
- old subprocess-based GUI glue is no longer the recommended path.

## Parallel work rules

Some PRs can be prepared in parallel after their dependencies are clear:

```text
PR 06 Qt shell can start after PR 01, but real scene editing needs PR 05.
PR 10 ML core can start after PR 01, but ML behaviour needs PR 03.
PR 12 web export should wait until PR 02 and PR 04 are stable.
```

Avoid parallel branches that modify the same architectural seam. For example, do not rewrite renderer ownership in one branch while another branch embeds the renderer in Qt.

## Recommended merge order

```text
00 docs/inventory
01 CMake engine library
02 loop-agnostic core
03 scene model
04 render command queue
05 scene JSON serialization
06 Qt editor shell
07 Qt scene editing
08 native runtime/player
09 Processing sketch layer
10 ML core primitives
11 ML behaviour demo
12 WASM/web export proof
13 retire Python editor glue
```

This order makes each later branch stand on an explicit foundation instead of depending on hidden assumptions.
