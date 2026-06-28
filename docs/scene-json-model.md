# Scene and JSON Model

## The three authoring styles

Bleue Mer can support three complementary authoring styles.

### 1. Processing-style sketch mode

A project can draw directly with immediate functions:

```cpp
void update(float dt)
{
    draw.background(Color::black);
    draw.stroke(Color::cyan);
    draw.circle(Vector3(0, 0, 0), 0.2f);
}
```

This mode is ideal for procedural art, math visualizations, simulations, fast experiments, and thesis-style demos.

### 2. Unity-style scene mode

A project can operate on persistent objects:

```cpp
Scene scene = SceneSerializer::Load("Assets/Scenes/Main.json");

void update(float dt)
{
    scene.update(dt);
    scene.render(renderer);
}
```

This mode is ideal for editor-authored projects, games, reusable assets, object inspection, collision, and component behavior.

### 3. Hybrid mode

A project can combine both:

```cpp
void update(float dt)
{
    draw.background(Color::darkGray);

    scene.update(dt);
    scene.render(renderer);

    draw.stroke(Color::yellow);
    draw.line(Vector3(-1, 0, 0), Vector3(1, 0, 0));
}
```

This hybrid model is the strongest identity for Bleue Mer: Processing-like expression plus Unity-like structure.

## What JSON should store

JSON should store persistent authored data:

- project metadata;
- active scene path;
- scene name and IDs;
- game objects;
- transform values;
- renderer configuration;
- collider configuration;
- rigidbody values;
- behavior references/configuration;
- ML model references and bindings;
- asset references.

JSON should not try to store every temporary immediate drawing command. Immediate drawing belongs to code. Persistent world objects belong to scene data.

## Proposed project file

```json
{
  "Version": 1,
  "Name": "dvd_bouncer",
  "ActiveScene": "Assets/Scenes/SampleScene.json",
  "Scenes": [
    "Assets/Scenes/SampleScene.json"
  ],
  "AssetRoots": [
    "Assets"
  ]
}
```

## Proposed scene file

```json
{
  "Version": 1,
  "ID": 0,
  "Name": "SampleScene",
  "Background": { "r": 0.02, "g": 0.02, "b": 0.04, "a": 1.0 },
  "GameObjects": [
    {
      "ID": 1,
      "Name": "Ball",
      "Enabled": true,
      "Transform": {
        "Position": { "x": 0.0, "y": 0.0, "z": 0.0 },
        "Rotation": { "x": 0.0, "y": 0.0, "z": 0.0 },
        "Scale": { "x": 0.1, "y": 0.1, "z": 1.0 }
      },
      "Renderer": {
        "Shape": "Circle",
        "Color": { "r": 1.0, "g": 1.0, "b": 1.0, "a": 1.0 }
      },
      "Rigidbody": {
        "Velocity": { "x": 0.5, "y": 0.25, "z": 0.0 }
      },
      "Collider": {
        "Type": "Circle",
        "Radius": 0.05
      },
      "Behaviours": [
        {
          "Type": "BounceBehaviour",
          "Config": {
            "BoundsMin": { "x": -1.0, "y": -1.0, "z": 0.0 },
            "BoundsMax": { "x": 1.0, "y": 1.0, "z": 0.0 }
          }
        }
      ]
    }
  ]
}
```

## Runtime loading flow

```text
ProjectSerializer::Load(ProjectData.json)
    -> project.ActiveScene
        -> SceneSerializer::Load(active scene path)
            -> Scene
                -> GameObject instances
                    -> component values
                    -> behavior instances/configuration
```

## Editor flow

```text
Open project
    -> load ProjectData.json
        -> load active Scene JSON
            -> populate Hierarchy
            -> show selected object in Inspector
            -> render Scene in Viewport
```

When the user edits a transform in the inspector:

```text
Inspector value changed
    -> modify GameObject.Transform in memory
        -> viewport updates immediately
            -> Save Scene writes JSON
```

## Processing draw and GameObject rendering should share a backend

Both immediate drawing and object rendering should become render commands:

```text
Processing API:
    draw.circle(...)
        -> RenderCommand(type=Circle)

GameObject Renderer:
    object.render(...)
        -> RenderCommand(type=Rect/Circle/Sprite/Mesh)
```

Then platform backends consume the same commands:

```text
Render commands
    -> native OpenGL backend
    -> Qt OpenGL viewport backend
    -> WebGL/WASM backend
```

This makes the engine portable and keeps the public API simple.
