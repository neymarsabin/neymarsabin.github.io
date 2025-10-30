---
title: "Tensor 5: Neuron Network Layers and Loss Functions"
date: 2025-10-15
layout: 'post'
---

Neuron initialization, layers and loss functions are important concepts in building neural networks.

<!--more-->

## Neuron Initialization
The default initialization weights of neurons usually depends on the type of layer we are using, Dense, Convolutional, and RNN, the Glorot Uniform initializer is a popular choice. It is also known as Xavier Uniform initializer. The default initializer is designed to keep the scale of the gradients roughly the equal in all layers.

**Biases** are usually initialized to zero except some cases of LSTM where specialization is required.

In keras, we can customize initializers by specifying the kernel and bias initializer arguments in the layer. 

```python
model = Sequential([
    Dense(128, activation='relu', kernel_initializer='he_normal', bias_initializer='zeros', input_shape=(784,)),
    Dense(64, activation='relu', kernel_initializer='he_normal', bias_initializer='zeros'),
    Dense(10, activation='softmax', kernel_initializer='glorot_uniform', bias_initializer='zeros')
])
```

Some common initialization methods are:
- **Zero Initialization**: all weights are zero, but it can lead to neurons learning the same thing.
- **Random Initialization**: weights to random values, scale of random values matters. Too high or too low can lead to vanishing or exploding gradients.
- **Xavier/Glorot Initialization**: weights based on the number of input and output neurons to keep the variance of activations and output of each layer approximately equal
- **He Initialization**: same like Xavier but designed for ReLu activation functions, considers only the number of inputs
- **Orthogonal Initialization**: weights of each layers are orthogonal matrices, to maintain the stability of network

Choice of initialization mostly depend on the type of activation function. Best way to determine the most effective initialization is to experiment with different methods and observing how quickly the network learns.

## Network Layers: Building a Hierarchy
A neural network consists of different types of layers, and their arrangment forms the network's architecture. A feedforward layer in keras is implemented using the `Dense` class.

```python
model.add(Dense(10, activation=’relu’))
```
The more nodes we have, the more data and time are needed to train the network. In a feedforward network, information flows from left to right, input layer to hidden layers to output layer.

### what is weight?
Weights are the parameters within the neural network that transform the input data within the network's layers. Each connection is represented by a weight, which is adjusted during training to minimize the error in predictions. Weights determine the importance of each input from the previous layer to the next layer.
Well, you can say an **important** node will have a higher weight, while a node which produces a zero weight all time is a useless node and is called **dead node**.

## The Optimizer and The Loss Function

```python
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```
Optimizer and loss functions are the two most important components of a training process. For each predicted output, we calculate how close we are the result i.e. loss value, and then change the weights using the optimizer to minimize the loss. Calculation of loss value is done using the loss function while adjusting the weights is done using the optimizer. 

### Loss Functions
The loss function provides the backpropagation algorithm with the necessary information to update the weights. The smaller the error, the better the model is at fitting the data. 

For most regression problems, we use `regression loss`. With problems related to probability we use one of the probabilistic loss functions. `Hinge Losses` is designed for `maximum-margin` classification problems. 

```python
# Either using a function handle
model.compile(optimizer=’adam’, loss=’mean_squared_error’)

# Or using the class
# Instantiate the class with desired configuration
mse_loss = MeanSquaredError(reduction=’auto’,
name=’mean_squared_error’)
# Compile the model using the class handle for the loss
model.compile(optimizer=’adam’, loss=mse_loss)
```

#### Mean-Squared Error (MSE)

```python
mse = tf.keras.losses.MeanSquaredError()
y_true = [3, -0.5, 2, 7]
y_pred = [2.5, 0.0, 2, 8]
mse_value = mse(y_true, y_pred).numpy()
print("Mean Squared Error:", mse_value)
```

#### Cosine-Similarity
Cosine similarity measures the similarity between two non-zero vectors. The advantage of using cosine similarity is that it is very easy and fast to compute. It is also scale invariant, meaning that it does not depend on the magnitude of the vectors.

```python
cosine_similarity = tf.keras.losses.CosineSimilarity(axis=-1)
y_true = [[1, 0, -1], [0, 1, 0]]
y_pred = [[-1, 0, 1], [0, -1, 0]]
cosine_sim_value = cosine_similarity(y_true, y_pred).numpy()
print("Cosine Similarity:", cosine_sim_value)
```

#### Binary Cross-Entropy
We use this for binary classification problems where the output has only two outcomes. The binary corss entropy function measures how far the model's predictions are from the actual labels.

```python
bce = tf.keras.losses.BinaryCrossentropy()
y_true = [0, 1, 1, 0]
y_pred = [0.1, 0.9, 0.8, 0.4]
bce_value = bce(y_true, y_pred).numpy()
print("Binary Cross-Entropy:", bce_value)
```

#### Categorical Cross Entropy
Used for multi-class outputs instead of a binary output. It is used when there are two or more label classes. Softmax is a suitable activation function to use with this loss function.

```python
cce = tf.keras.losses.CategoricalCrossentropy()
y_true = [[1, 0, 0], [0, 1, 0], [0, 0, 1]]
y_pred = [[0.9, 0.05, 0.05], [0.1, 0.8, 0.1], [0.2, 0.2, 0.6]]
cce_value = cce(y_true, y_pred).numpy()
print("Categorical Cross-Entropy:", cce_value)
```

### Network Optimizers
Iteratively refining the model parameters to improve model accuracy and reduce error. The optimizer works on the background process and its main objective is to minimize the loss function by updating the weights of the network. The optimizer uses search algorithms to find the estimate the next set of weight values based on the loss function. `Gradient Descent` is the most common optimization algorithm.

Okay, the book describes a good example. Suppose you are lost on a trek on a hill. One strategy to find bottom of the hill is to look around and find the steepest slope, then check direction again and repeat until we get stuck again. We need two parameters here: the number of steps before we check for direction and the direction of descent at each step.

Choosing a small step would slow down convergence, while a large step might overshoot the minimum.
With keras, we can use basic and adaptive gradient descent optimizers. We have mini batch gradient descent and Adam optimizer. Adam is considered best among the adaptive optimizers.

```python
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```

