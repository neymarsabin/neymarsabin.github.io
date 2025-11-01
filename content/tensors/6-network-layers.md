---
title: "Tensor 6: Network Layers"
date: 2025-10-30
layout: 'post'
---

Dense, Normalization, Dropout, Pooling, Convolutional Layers 

<!--more-->
The layers in a neural network are the building blocks that are fundamental in extracting/transforming features from input data each serving a distinct purpose in the learning process. 

## Dense (Fully Connected) Layers
Can be used as an input layer, hidden layer or output layer.

```python
model = Sequential()
# input layer
model.add(Dense(units=64, activation='relu', input_shape=(input_dim=100,)))
# hidden layer
model.add(Dense(units=32, activation='relu'))
# output layer
model.add(Dense(units=10, activation='softmax'))
```
- `number of inputs(Neurons)` -> unit parameters in dense layer specifies the number of neurons in that layer, is generally determined through experiments.
- `activation function` -> commonly used in hidden layers, ReLU is commonly used that helps vanishing gradient problem
- `input dimensions` -> `input_dim` parameter is used to specify the shape of the input data, typically assumed to be dynamic
- `output layer configuration` -> configuration for the output layer depends on the specific problem(e.g. number of classes for classification tasks), softmax is common


## Normalization Layers
Normalizes the inputs of each mini batch to have a mean of 0 and a standard deviation of 1.

- `Batch Normalization` -> normalizes the inputs across the batch dimension, helps stabilitze and accelerate training in each batch, 
- `Layer Normalization` -> normalizes across each features or each layers, computes the mean and variance for each layers

```python
model.add(Dense(64))
model.add(LayerNormalization())
model.add(Activation('relu'))
```

## Dropout Layers
The dropout layer randomly sets a fraction of input units to 0 at each update during training time, which helps prevent overfitting. `rate = 0.2` means 20% of the input units will be dropped or their weight will be set to zero during training. A rate too high may lead to underfitting, while a rate too low may not effectively prevent overfitting. Commonly applied to intermediate layers.

```python
model = Sequential()
mode.add(Dense(64, activation='relu', input_shape=(input_dim,)))
model.add(Dropout(0.2)) ## applying 20% dropout
model.add(Dense(10, activation='softmax'))
```

