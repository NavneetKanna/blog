---
layout: post
title: "Deep Learning"
date: 2023-12-25
---

Before delving into the realms of deep learning, it's essential to understand its fundamental aspects: What is it? What problems does it aim to solve? 

## What is Deep Learning?

Deep learning can be defined in two ways: verbally and mathematically.

### Verbal Definition

To put it simply,

> It is the study of enabling computers/algorithms to learn from experience without explicit programming.

Let's take for example writing an algorithm to predict handwritten digits. If we were to manually write an algorithm, there would be a lot of if else statements:

```c
if (there is a curve on the top) {
    printf("maybe it is 6, 3, 2, 9");
}

else if (there are two lines) {
    printf("maybe it is 1, 7");
}
```

This is just pseudo code, imagine writing the actual algorithm. There should be a much better, efficient and intelligent way of doing this, which is, of course, deep learning. 

Given sufficient data for training, DL algorithms can learn the underlying patterns and be able to generalise to new data. So for example, the algorithm learns the pattern that if there are two curves on top of each other, then it is most likely an eight, and so on. And now given any new handwriting of eight that it has not seen, it should be able to predict the correct number.

### Mathematical Definition

The main idea is, a neural network tries to approximate a function. Given a dataset of inputs and outputs, we assume that there was some underlying function that produced the output values given the input values and its the goal of a neural network to approximate this function as close as possible.

Given some function $$ y = f^*(x) $$ a neural network defines a mapping $$ y = f(x) $$ which is a close approximation to $$ f^* $$.

The branch of mathematics related to this is called Function approximation. In the mathematical theory of artificial neural networks, there is a theorem called Universal approximation theorem, which states that

> A neural network comprising of an output layer and at least one hidden layer with an activation function can approximate any function.

An important point to note here is that, neural networks are universal approximators, ie, they can be used to approximate any kind of function.

So, now we know, what is machine learning and what problem does deep learning solve. With the basics understood, let us now move ahead with understanding forward propogation.

## Forward Propogation 

We all know the general flow of a neural network: inputs are multiplied with the weights and bias to get an output, then we use a loss function to get the error between the prediction and the true value, finally we backpropagate this error to adjust the weights and bias.

For the forward propagation part, I am assuming you know the basics, however, to grasp it fundamentally, it is important to understand the *why's* of the below topics

### Activation function

![feedforward_no_activation]({{ "/assets/images/feedforward_no_activation.svg" | prepend: site.baseurl }})

If we dont have activations functions, or in other words, a non-linear function, the forward propagation would like this 

```math
z1 = w1*x1 + b 
z2 = w2*x1 + b 
z3 = (w3*z1 + w5*z2) + b 
z4 = (w4*z1 + w6*z2) + b 
o  = (w7*z3 + w8*z4) + b 
```

The final output is just a linear weighted sum of the form  *y = mx + b*. This is because a combination of linear functions is a linear function itself, which is just a straight line, and hence the model is not really useful if it has to learn a funtion where there is a non-linear (other than a straight line) mapping between the inputs and outputs, which is for most of the real world cases. 

### Bias 

Again lets consider the formula without bias

```math
z1 = w1*x1
```

This is of the form y = mx, this is a straight line through the *origin*, without the bias, the activation function would be restricted to pass through the origin. Hence, by using bias we can shift the activation function which is important.

## Backpropagation

Okay, let's recap. We have some inputs that propagate forward through hidden layers, getting multiplied by weights and having biases added along the way, until they finally reach the output layer. 

### Loss Function

The loss function is used to tell how well the model is performing and it is this function that we are going to minimise during training.

### Backpropagation or Gradient Descent or Automatic Differentiation ? 

#### Backpropogation

Backpropagation is the process of calculating the gradients of a loss function with respect to the weights.

*Derivative and gradient*

Derivative of a function tells us how to change inputs to the function in a way that increases or decreases the output of the function, so we can get closer to the minimum or maximum of the function [source](https://machinelearningmastery.com/gradient-in-machine-learning/).

Basically, the derivative of a function at a point tells us the direction of the steepest ascent, and taking the negative of that value will give the direction of the steepest descent, which is what we want. 

A gradient is the derivative of a multivariable function, which a loss function is.

Therefore, backpropagation is an algorithm that calculates the gradients of the loss function.

#### Gradient Descent

All that backpropagation does is, it calculates the gradient (the direction and magnitude of the steepest descent), but this gradient needs to be applied to the  
model parameters to move them towards the minimum of the function. This process is called gradient descent.

*Recap*

- Inputs are multiplied by the weigths during the forward pass.
- Loss function is used to calculate the error.
- Gradients are calculated using backpropagation alorithm.
- The weights are adjusted by the taking the negative of the gradient in a step called gradient descent.

### Automatic differentiation (AD)

There are different ways of computing derivatives, numerical, symbolic and AD. The first two have [disadvantages](https://www.jmlr.org/papers/volume18/17-468/17-468.pdf) of its own and cannot be used in deep learning.

AD is an efficient/algorithmic way to compute the derivates during the backpropagation process. It uses computational graphs.
