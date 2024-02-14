---
layout: post
title: "Part 2: Feedforward Networks"
date: 2024-01-01
---

We all know the general flow of a neural network: inputs are multiplied with the weights and bias to get an output, then we use a loss function to get the error between the prediction and the truth value, finally we backpropagate this error to adjust the weights and bias.

For the forward propagation part, I am assuming you know the basics, but below are some important questions that are import to understand the fundamentals

## Activation function

![feedforward_no_activation]({{ "/assets/images/feedforward_no_activation.svg" | prepend: site.baseurl }})

If we dont have activations functions, or in other words, a non-linear function, the forward propagation would like this 

```math
z1 = w1*x1 + b 
z2 = w2*x1 + b 
z3 = (w3*z1 + w5*z2) + b 
z4 = (w4*z1 + w6*z2) + b 
o  = (w7*z3 + w8*z4) + b 
```

The final output is just a linear weighted sum of the form  *y = mx + b*. This is because a combination of linear functions is a linear function itself, which is just a straight line, and hence the model is not really useful if it has to learn a funtion where there is a non-linear (other than a straight line) mapping between the inputs and outputs. 

## Bias 

Again lets consider the formula without bias

```math
z1 = w1*x1
```

This is of the form y = mx, this is a straight line through the *origin*, without the bias the activation function would be restricted to pass through the origin. Hence, by using bias we can shift the activation function which is important.