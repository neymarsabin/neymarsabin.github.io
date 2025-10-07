---
title: "Tensor 3: Components of Neural Networks"
date: 2025-10-07
layout: 'post'
---

We look at the components of a neural network in this post. Lets try to understand what each component does and how they contribute to make intelligent decisions.

<!--more-->

We already know that there are input/output layers of neurons and one or more hidden layers. What are other key components?

## Neurons: Building Blocks
Artificial neurons are the basic units of neural network. It is like a biological neuron which receives input signals, processes them and produces an output signal. Each neuron has multiple inputs and one output.

- **Input Weights** -> each neuron has multiple inputs from other neurons or external data sources with individual weights and during training these weights are adjusted to minimize the error.
- **Activation Function** -> after receiving weighted inputs, a neuron applies an activation function to introduce non-linearity. the output from the activation function is then passed to the next neuron as input.
- **Bias Term** -> each neuron has a bias term that allows the network to account for possible offsets or biases in the data, provides more flexibility by allowing the network to fit the data more accurately.

## Some Activation Functions
### Sigmoid Function
- Formula: \( \sigma(x) = \frac{1}{1 + e^{-x}} \)
- Range: (0, 1)
- Use Case: Commonly used in binary classification problems.
- one drawback is that it is prone to the vanishing gradient problem, where gradients become very small during backpropagation, making it difficult for the network to learn.

### Tanh Function
- Formula: \( \tanh(x) = \frac{e^{x} - e^{-x}}{e^{x} + e^{-x}} \)
- Range: (-1, 1)
- Use Case: Often used in hidden layers of neural networks.
- zero centered, which can help with convergence during training.
- it can also suffer from the vanishing gradient problem.

### ReLU (Rectified Linear Unit)
- Formula: \( \text{ReLU}(x) = \max(0, x) \)
- Range: [0, ∞)
- most widely used activation function in deep learning, especially in hidden layers.
- helps maintain strong gradients by maintaining a nonzero gradient for positive inputs
- can suffer from the "dying ReLU" problem, where neurons can become inactive and only output zero for all inputs.

### Leaky ReLU
- Formula: \( \text{Leaky ReLU}(x) = \max(0.01x, x) \)
- Range: (-∞, ∞)
- Use Case: Used to address the "dying ReLU" problem.
- introduces a small gradient when the unit is not active, allowing the neurons to never die.

## But Why Activation Functions?
When an input signal is received at a node, it generates an output y using the formula:

\[ y = f\left(\sum (w_i \cdot x_i) + b\right) \]

where:
- \( x_i \) are the input signals like x1, x2, x3....
- \( w_i \) are the weights associated with each input signal.
- \( b \) is the bias term.

The node sums the product of the weights and the input values, add a number, a bias and then apply the activation function f to the weighted sum to produce the output y.
