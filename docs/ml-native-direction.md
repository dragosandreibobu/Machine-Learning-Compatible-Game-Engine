# ML-Native Engine Direction

## Vision

Bleue Mer can have a distinctive identity by treating machine learning as a native engine capability rather than an external Python-only workflow.

The goal is not to clone all of scikit-learn. The goal is to implement a focused set of understandable C++ ML primitives that can be used directly by scenes, objects, simulations, and editor tools.

## What ML-native means

ML-native means:

- models are implemented in C++;
- models can run inside the engine update loop;
- models can be attached to game objects as behaviours/components;
- model configuration can be serialized;
- the editor can inspect model inputs, outputs, and training data;
- web export can run the same model code through WASM.

ML-native does **not** mean:

- the runtime depends on Python;
- every scikit-learn feature must be reimplemented;
- training must be complex from the beginning;
- gameplay must block on heavyweight ML workflows.

## First algorithms to implement

Start with small, understandable algorithms:

1. Linear Regression
2. Logistic Regression
3. K-Nearest Neighbors
4. K-Means
5. Perceptron
6. Simple Decision Tree
7. Tiny feed-forward neural network

These algorithms are useful for demos, visualizations, and game/simulation behavior while remaining feasible to implement cleanly in C++.

## Suggested ML module

```text
Engine/ML/
  Tensor
  Matrix
  Dataset
  Model
  SupervisedModel
  UnsupervisedModel
  LinearRegression
  LogisticRegression
  KNearestNeighbors
  KMeans
  DecisionTree
  ModelSerializer
  MLBehaviour
```

## Runtime API sketch

```cpp
ML::LinearRegression model;
model.fit(xTrain, yTrain);

float prediction = model.predict({ ballX, ballY, paddleY })[0];
```

For an engine object:

```cpp
class MLBehaviour : public Behaviour
{
public:
    std::unique_ptr<ML::Model> model;
    InputBinding inputs;
    OutputBinding outputs;

    void update(GameObject& self, Scene& scene, float dt) override
    {
        auto x = inputs.collect(self, scene);
        auto y = model->predict(x);
        outputs.apply(self, scene, y);
    }
};
```

## Example: ML-controlled paddle

A strong first demo would be an ML-controlled paddle in the existing bouncer-style project.

Inputs:

```text
ball.position.x
ball.position.y
ball.velocity.x
ball.velocity.y
paddle.position.y
```

Output:

```text
paddle.rigidbody.velocity.y
```

This demo would connect the whole vision:

```text
GameObject scene model
    + Processing-like visualization
    + native C++ ML prediction
    + editor-inspectable data
    + possible web export
```

## Model serialization

A model file can start as JSON:

```json
{
  "Version": 1,
  "Type": "LinearRegression",
  "Inputs": [
    "ball.position.x",
    "ball.position.y",
    "ball.velocity.x",
    "ball.velocity.y",
    "paddle.position.y"
  ],
  "Outputs": [
    "paddle.rigidbody.velocity.y"
  ],
  "Weights": [0.2, -0.4, 0.7, 0.1, 0.9],
  "Bias": 0.0
}
```

This keeps ML assets aligned with scene/project serialization.

## Editor support

The editor can eventually include an ML panel with:

- dataset viewer;
- model asset inspector;
- input/output binding editor;
- train/evaluate buttons;
- loss plot;
- decision boundary visualization;
- cluster visualization;
- model debug output during play mode.

## Relationship to Processing-style projects

Processing-style drawing is excellent for ML visualization:

- draw regression lines;
- draw clusters;
- draw loss curves;
- draw decision boundaries;
- draw agent observations;
- draw model predictions.

Unity-style GameObjects are excellent for ML behavior:

- attach an ML model to an enemy;
- control a paddle;
- classify world states;
- select behaviors;
- drive procedural simulations.

JSON is excellent for persistence:

- store model parameters;
- store model bindings;
- store dataset references;
- store scene objects using ML behaviours.

Together, these form a coherent ML-native engine model.
