---
title: "Tensor 1: Using Tensors"
date: 2025-09-15
layout: 'post'
---

As the AI models grew larger and datasets more expansive, the need for parallel computing has risen. Deep learning models based on their scale needs to represent and process data across multiple dimensions. This is where tensors are crucial.
<!--more-->

> Tensors are mathematical structures that generalize the concept of vectors and matrices to higher dimensions, enabling efficient computations over large-scale data.

So, what is the difference between tensors and standard arrays?
They are designed to perform highly complex numerical operations, using tensors allows deep learning tasks that require intensive calculations across millions of parameters.


Let's compare tensors with numpy arrays:
> If you don't know numpy, it's a powerful library in Python for numerical computing, providing support for arrays and matrices along with a collection of mathematical functions to operate on these data structures.

- **GPU Support**: NumPy arrays are primarily designed for CPU computations, while tensors are optimized for parallel computing, they can efficiently distribute and run computations across GPUs and TPUs
- **Gradient Computation**: NumPy lacks ability to compute gradients, which is essential for training deep learning models.
- **Eager Execution**: Tensorflow supports both eager execution and graph execution
- **Broadcasting**: NumPy array and tensor support broadcasting. This allows operations on arrays of different shapes.

### Is there a way to convert numpy arrays to tensors?
Well yes, you can convert using TensorFlow. Let's look at an example:
```python
import numpy as np
import tensorflow as tf

# create a numpy array
np_array = np.array([1, 2, 3, 4, 5])
# convert to tensor
tensor_from_np = tf.convert_to_tensor(numpy_array)
# or we can do
# tensor_from_np = tf.constant(numpy_array)
print(tensor_from_np)

# convert tensor to numpy array
tensor_to_np = tensor_from_np.numpy()
print(tensor_to_np)

# create a tensor directly
tensor = tf.constant([[1, 2], [3, 4], [5, 6]])
print(tensor)
print(tensor.shape)  # Output: (3, 2)

# convert tensor to numpy array
tensor_to_np_new = tensor.numpy()
print(tensor_to_np_new)
```

### Tensor Operations
Now that we know tensor is a multi-dimensional array, let's look at some common operations we can perform on tensors.

#### Creating Tensors
```python
import tensorflow as tf

# creating a constant tensor
tensor_a = tf.constant([[1, 2], [3, 4]])
print(tensor_a)

# creating a variable tensor
tensor_b = tf.Variable([[5, 6], [7, 8]])
print(tensor_b)
```

#### Mathematical Operations
Supports addition, substraction, multiplication, division etc.
```python
# Element-wise addition
tensor_sum = tf.add(tensor_a, tensor_b)
print("Sum:\n", tensor_sum)
```

#### Reshaping Tensors
Reshaping and slicing tensors is common in deep learning.
```python
## reshaping a tensor from one shape to another while keeping the total number of elements the same.
print("Original shape:", tensor_a.shape)

## reshape to (4, 1)
reshaped_tensor = tf.reshape(tensor_a, (4, 1))
```

## Sense of Parallelism with Tensorflow
Optimizing our code and utilizing tensorflow's built in capabilities is crucial. Avoid manual computations using loops. Instead leverage tensor operations that are optimized for parallel execution on GPUs and TPUs. For example batch matrix multiplication - common operation in deep learning.

```python
tensor_1 = tf.random.normal([10, 20, 30])
tensor_2 = tf.random.normal([10, 30, 40])

## if you want to list or check GPUs available
gpus = tf.config.experimental.list_physical_devices('GPU')
print("GPUs available: ", gpus)

# Batch MATRIX multiplication
# tensorflow will handle the parallelism internally with matrix multiplications
# in parallel across the batch dimension
result_batch_mul = tf.matmul(tensor_1, tensor_2)
print("Result: ", result_batch_mul)
print("Result shape:", result_batch_mul.shape)

```
There are various ways to do parallelism in tensorflow. We will explore more as we move forward.
