---
layout: post
title: "Part 2: Feedforward Networks"
date: 2024-01-01
---

## Activation function

1. If we dont have activations functions, or in other words, a non-linear function, the forward propagation would like this 

```math
z1 = w1*x1 + b 
z2 = w2*x1 + b 
z3 = (w3*z1 + w5*z2) + b 
z4 = (w4*z1 + w6*z2) + b 
o  = (w7*z3 + w9*z4) + b 
```

The final output is just a linear weighted sum of the form  *y = mx + b*. This is because a combination of linear functions is a linear function itself, which is just a straight line, and hence the model is not really useful if it has to learn a funtion where there is a non-linear (other than a straight line) mapping between the inputs and outputs. 