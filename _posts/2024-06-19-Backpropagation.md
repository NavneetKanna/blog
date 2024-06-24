---
layout: post
title: "Part 3: Backpropagation"
date: 2024-02-22
---

Okay, let's recap. We have some inputs that propagate forward through hidden layers, getting multiplied by weights and having biases added along the way, until they finally reach the output layer. 

## Loss Function

The loss function is used to tell how well the model is performing and it is this function that we are going to minimise during training.

## Backpropagation or Gradient Descent or Automatic Differentiation ? 

### Backpropogation

Backpropagation is the process of calculating the gradients of a loss function with respect to the weights.

*Derivative and gradient*

Derivative of a function tells us how to change inputs to the function in a way that increases or decreases the output of the function, so we can get closer to the minimum or maximum of the function [source](https://machinelearningmastery.com/gradient-in-machine-learning/).

Basically, the derivative of a function at a point tells us the direction of the steepest ascent, and taking the negative of that value will give the direction of the steepest descent, which is what we want. 

A gradient is the derivative of a multivariable function, which a loss function is.

Therefore, backpropagation is an algorithm that calculates the gradients of the loss function.

### Gradient Descent

All that backpropagation does is, it calculates the gradient (the direction and magnitude of the steepest descent), but this gradient needs to be applied to the  
model parameters to move them towards the minimum of the function. This process is called gradient descent.

*Recap*

- Inputs are multiplied by the weigths during the forward pass.
- Loss function is used to calculate the error.
- Gradients are calculated using backpropagation alorithm.
- The weights are adjusted by the taking the negative of the gradient in a step called gradient descent.

### Automatic differentiation 

Autodiff is an efficient way to compute the derivates during the backpropagation process. It uses computational graphs.