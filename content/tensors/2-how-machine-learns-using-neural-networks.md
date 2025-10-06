---
title: "Tensor 2: How Machine Learns Using Neural Networks"
date: 2025-10-06
layout: 'post'
---

Did you know that a feedforward neural network with a single hidden layer can approximate any continuous function to arbitary precision, given the right activation function and enough neurons in the hidden layer? This is known as the Universal Approximation Theorem.

<!--more-->

This means that in theory, a machine can theoretically learn any function, no matter how complex and produce almost any output from the input set of data. That's where machine learning algorithms come in. They are different than classical programming where we used to write rules and logic to get the desired output. In machine learning, we provide a large set of input-output pairs and let the algorithm learn the underlying patterns and relationships.

The book provides a good example:
Let's take input numbers 0, 1, 2 to 10 and the corresponding set of output numbers 0, 1, 4, 9, 16, 25, 36, 49, 64, 81 and 100. The task is to predict the output number for an input number 11.

```python
# a normal implementation of this problem would be:
def predict(x): 
    return x*x
print('output(11) = ', predict(11))
# output(11) =  121
```

We guess the function is x -> x*x. This is called classical programming. But in case of machine learning, we would provide the input-output pairs to the algorithm and let it learn the underlying function.

- create and configure a neural network "supervised learning"
- feed the neural network with input-output pairs like {0,0}, {1,1}, {2,4}, {3,9} ... {10,100}
- train the neural network
- use the trained neural network to predict the output using 11 as an input

The neural networks interact with each other to produce a function which would match the output values as much as possible from the set of input data. It does this by updating the weight of each node to reduce the value of the error function (Loss function). The loss function is a measure of how far the predicted output is from the current output. The goal of training is to minimize the loss function.

The book does not provide a code example for this, but I tried something on my own using tf:

```python
# first prepare the data
x = np.array([i for i in range(11)], dtype=float) # classic numpy array definition
y = np.array([i*i for i in range(11)], dtype=float) # y = x*x values in numpy array

# defining a sequential model with relu activation function
# input shape is 1 dim
model = tf.keras.Sequential([
    tf.keras.layers.Dense(8, activation='relu', input_shape=[1]),
    tf.keras.layers.Dense(8, activation='relu'),
    tf.keras.layers.Dense(1)
])

# adam optimizer and loss calculation method is straight forward mean squared error
model.compile(optimizer='adam', loss='mean_squared_error')
model.fit(x, y, epochs=5000, verbose=0) # running for 500 epochs but lets see

# 5. Predict new value
test_input = np.array([11.0])
prediction = model.predict(test_input)
print(f"Predicted output for input {test_input[0]} = {prediction[0][0]:.2f}")
```

Within `5000` epochs, the model predicts the output for input `11` as `113.11`. This is still not perfect but you get my point, right?

To be continued.....
