# Bleue Mer

Bleue Mer is a bachelor-thesis-era game engine/editor prototype exploring a compact C++ runtime, a Unity-inspired object model, and a Processing-like creative coding API. The current repository contains:

- a C++ OpenGL/GLUT engine layer with math primitives, render helpers, input, and `GameObject` abstractions;
- Python/PyQt editor prototypes for project creation, scene browsing, hierarchy display, assets, terminal, inspector, and scene view panels;
- several experimental C++ projects and demos.

The project is best understood as a prototype for a future architecture rather than a finished engine. Its strongest direction is a C++ engine/editor stack where scenes are authored as data, loaded into runtime `GameObject`s, and rendered either natively or on the web.

## Documentation map

Start with these documents:

1. [Architecture Vision](docs/architecture-vision.md) — the high-level direction for the engine, editor, data model, and web deployment.
2. [Scene and JSON Model](docs/scene-json-model.md) — how Processing-style code, Unity-like objects, and JSON scene files fit together.
3. [Runtime Loop Ownership](docs/runtime-loop-ownership.md) — how Qt, native runtime, and browser hosts should drive the engine without the engine owning the main loop.
4. [ML-Native Engine Direction](docs/ml-native-direction.md) — how native C++ machine-learning primitives could become first-class engine components.
5. [Modernization Roadmap](docs/modernization-roadmap.md) — an implementation sequence for turning the prototype into a coherent next-generation system.

## Current architectural center

The current C++ engine already has the vocabulary of the future system:

- `Vector3` and `Color` define basic value types.
- `RenderEngine` exposes a small immediate drawing API.
- `GameEngine::GameObject` owns component-like data: `Transform`, `Rigidbody`, `Renderer`, and `Collider`.
- input is abstracted behind `GameEngine::Input`.

The current editor prototype already points toward the authoring side:

- projects contain `ProjectData.json`;
- scenes are placed under `Assets/Scenes`;
- the hierarchy reads scene JSON and displays scene/game-object names;
- the project page sketches a Unity-like editor layout.

The next architectural milestone is to make these two halves share a formal scene model.

## Recommended future identity

> Bleue Mer should become a C++/Qt game and simulation editor with a Processing-like drawing layer, Unity-like `GameObject` behavior, JSON-authored scenes, native C++ ML components, and eventual WebAssembly/WebGL deployment.

That identity keeps the original elegance of the engine while giving the project a clear path forward.
