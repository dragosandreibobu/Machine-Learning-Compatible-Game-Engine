# Runtime Loop Ownership

## Core rule

The engine should not own the main loop.

Instead:

```text
Host owns loop.
Engine exposes frame functions.
```

This is essential because Bleue Mer may run in at least three hosts:

1. C++/Qt editor;
2. native standalone runtime;
3. browser/WebAssembly runtime.

Each host has different loop rules.

## Current prototype constraint

The current renderer is GLUT-centered. It creates the window, registers callbacks, invokes the start callback, and enters `glutMainLoop()`. That is suitable for demos but too rigid for Qt and browser deployment.

## Desired engine API

The engine should expose something like:

```cpp
class Engine
{
public:
    void initialize();
    void shutdown();

    void update(Scene& scene, float deltaTime);
    void render(Scene& scene, Renderer& renderer);
};
```

The host decides when to call these functions.

## Qt editor loop

In editor mode, Qt owns the loop:

```cpp
int main(int argc, char** argv)
{
    QApplication app(argc, argv);

    EditorContext context;
    MainWindow window(&context);
    window.show();

    return app.exec();
}
```

A `QTimer` or viewport update event can drive frames:

```cpp
connect(timer, &QTimer::timeout, this, [this]()
{
    float dt = clock.deltaTime();

    if (editorContext.isPlaying)
    {
        engine.update(editorContext.scene, dt);
    }

    sceneViewport->update();
});
```

The `QOpenGLWidget` performs rendering:

```cpp
void SceneViewport::paintGL()
{
    renderer.beginFrame();
    engine.render(editorContext.scene, renderer);
    renderer.endFrame();
}
```

## Native runtime loop

A standalone runtime can own a traditional game loop:

```cpp
while (window.isOpen())
{
    float dt = clock.deltaTime();

    input.poll();
    engine.update(scene, dt);

    renderer.beginFrame();
    engine.render(scene, renderer);
    renderer.endFrame();
}
```

This host may use GLFW, SDL, Qt, or another backend. The engine core should not care.

## Web loop

In the browser, the browser owns the event loop. With Emscripten, the host would register a frame callback:

```cpp
emscripten_set_main_loop(frame, 0, true);
```

Then each browser frame calls:

```cpp
void frame()
{
    float dt = webClock.deltaTime();

    input.updateFromBrowser();
    engine.update(scene, dt);
    engine.render(scene, renderer);
}
```

The engine remains the same. Only the host changes.

## Why this matters

Loop ownership determines whether the engine can be embedded, tested, exported, and reused.

If the engine owns `mainLoop()`, it is hard to embed in Qt and hard to port to the browser.

If the host owns the loop, the same engine can run in:

```text
Qt editor
native game executable
browser canvas
unit tests
headless simulations
ML training/evaluation tools
```

## Recommended transition path

1. Keep existing GLUT demos working as a legacy host.
2. Extract engine update/render functions from GLUT callbacks.
3. Make GLUT call those functions instead of owning all engine behavior.
4. Add a Qt viewport host.
5. Add a standalone native host.
6. Add a web host later.

The important refactor is not replacing GLUT immediately. The important refactor is making GLUT just one host among several.
