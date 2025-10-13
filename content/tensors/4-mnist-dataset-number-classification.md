---
title: "Tensor 4: MNIST Dataset - Number Classification"
date: 2025-10-13
layout: 'post'
---

Let's explore MNIST dataset and make a binary classifier. We will only use the output node to check if the handwritten digit is 8.

<!--more-->

We are trying to classify `8` from test datasets, here is our combination:
- Input Layer: 784 nodes (28x28 pixels)
- Hidden Layer: 784 nodes, Activation Function: ReLU
- Output Layer: 1 node, Activation Function: Sigmoid
- Loss Function: Binary Cross Entropy
- Optimizer: Adam Optimizer

```python
import numpy as np
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense

mnist = tf.keras.datasets.mnist
(xtrain, ytrain), (xtest, ytest) = mnist.load_data()
xtrain_flattened = xtrain.reshape(len(xtrain), 28*28)
xtest_flattened = xtest.reshape(len(xtest), 28*28)

y_train = np.where(ytrain != 8, 0, 1)
y_test = np.where(ytest != 8, 0, 1)

model = Sequential()
model.add(Dense(784, activation='relu', input_shape=(784,)))
model.add(Dense(784, activation='relu'))
model.add(Dense(1, activation='sigmoid'))
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

# train and evaluate the model, running for 5 epochs
model.fit(xtrain_flattened, y_train, epochs=5)
y_predict = model.predict(xtest_flattened)

# evaluate the model
model.evaluate(xtest_flattened, y_test)
```

The accuracy of the model is around 98.5% after 5 epochs. Here we just tried a combination of layers and activation functions. You can try different combinations to see how the accuracy changes.

