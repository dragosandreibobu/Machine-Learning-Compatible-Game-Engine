# Architecture Vision

## One-sentence direction

Bleue Mer should evolve into a C++/Qt game and simulation editor built around a loop-agnostic C++ engine core, a Processing-like immediate drawing API, a Unity-like `GameObject` scene model, JSON serialization, native ML components, and eventual web deployment through WebAssembly/WebGL.

## Why this direction fits the existing project

The current codebase already contains two compatible halves:

1. **C++ runtime and engine vocabulary**
   - math primitives;
   - color primitives;
   - immediate render helpers;
   - input abstraction;
   - `GameObject` with component-like fields.

2. **Python/PyQt editor prototype vocabulary**
   - project creation;
   - project data;
   - scene files;
   - hierarchy panel;
   - scene view;
   - assets panel;
   - inspector concept;
   - run/build action.

The missing bridge is not conceptual. The missing bridge is a formal shared runtime/editor model:

```text
ProjectData.json
    -> active scene JSON
        -> C++ SceneLoader
            -> Scene
                -> GameObjects
                    -> Components
                        -> rendering, physics, input, ML behavior
```

## Target dependency direction

The clean dependency direction is:

```text
EditorQt  ---> Engine
Runtime   ---> Engine
WebHost   ---> Engine
Engine    -X-> EditorQt
Engine    -X-> Runtime
Engine    -X-> WebHost
```

The engine should not know whether it is running inside a Qt editor, a native executable, or a browser canvas. It should expose initialization, update, render, serialization, and component APIs. Hosts decide when and where to call those APIs.

## Core layers

A future version could be organized like this:

```text
Engine/
  Core/
    Application
    Scene
    GameObject
    Component
    Behaviour
    Time
  Math/
    Vector2
    Vector3
    Color
    Matrix
  Rendering/
    Renderer
    RenderCommand
    PrimitiveRenderer
    OpenGLBackend
    WebGLBackend
  Input/
    Input
    KeyCode
    Mouse
  Serialization/
    ProjectSerializer
    SceneSerializer
    ModelSerializer
  ML/
    Dataset
    Model
    LinearRegression
    KMeans
    DecisionTree
    MLBehaviour

EditorQt/
  MainWindow
  SceneViewport
  HierarchyPanel
  InspectorPanel
  AssetsPanel
  ConsolePanel
  MLPanel

Runtime/
  main.cpp
  NativeWindow

Web/
  index.html
  wasm bootstrap
  asset manifest
```

## Engine core responsibilities

The engine core should own:

- scene data;
- game object identity and hierarchy;
- component data;
- behavior update ordering;
- render command generation;
- input state representation;
- serialization of engine-owned data;
- optional ML model execution.

The engine core should **not** own:

- the Qt application loop;
- the browser event loop;
- editor widgets;
- platform-specific window creation policy;
- editor-only selection state, gizmo state, or dock layout.

## Editor responsibilities

The C++/Qt editor should own:

- project opening/creation UI;
- hierarchy display;
- inspector editing;
- asset browsing;
- scene viewport hosting;
- edit/play/pause state;
- saving and loading scenes through engine serializers;
- visual tools for ML datasets/models;
- build/export commands.

The editor should manipulate actual engine `Scene` and `GameObject` instances in memory, not constantly shell out to separate helper programs.

The previous subprocess-heavy GUI communication model should be treated as prototype glue. A C++/Qt rewrite should centralize project, scene, asset, and build operations behind typed services instead of routing internal editor state through command-line arguments and parsed stdout.

## Runtime responsibilities

A standalone runtime should own:

- creating a native window;
- loading a project/scene;
- running the game loop;
- forwarding input into the engine;
- calling `engine.update(dt)` and `engine.render()`;
- packaging assets for native distribution.

## Web host responsibilities

The web host should own:

- loading the WASM module;
- fetching or mounting scene/assets files;
- connecting browser input to engine input;
- using the browser animation loop;
- rendering through WebGL or a WebGL-compatible backend.

## Architectural rule of thumb

If a feature is needed by editor, runtime, and web deployment, it belongs in `Engine`.

If a feature is only a tool for authoring, visualization, docking, selection, or editing workflow, it belongs in `EditorQt`.

If a feature only starts a platform-specific executable/window/canvas, it belongs in a host layer.
