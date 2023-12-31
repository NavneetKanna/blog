---
layout: post
title: "Part 1:  What is machine learning ?"
date: 2023-12-25
---

Before delving into the realms of machine learning, it's essential to understand its fundamental aspects: What is it? What problems does it aim to solve? 

## What is Machine Learning?

Machine learning can be defined in two ways: verbally and mathematically.

### Verbal Definition

To put it simply,

> Machine learning is the study of enabling computers/algorithms to learn from experience without explicit programming.

Let's take for example writing an algorithm to predict handwritten digits. If we were to manually write an algorithm, there would be a lot of if else statements:

```c
if (there is a curve on the top) {
    printf("maybe it is 6, 3, 2, 9");
}

else if (there are two lines) {
    printf("maybe it is 1, 7");
}
```

This is just pseudo code, imagine writing the actual algorithm. There should be a much better, efficient and intelligent way of doing this, which is, of course, machine learning. 

Given sufficient data for training, ML algorithms can learn the underlying patterns and be able to generalise to new data. So for example, the algorithm learns the pattern that if there are two curves on top of each other, then it is most likely an eight, and so on. And now given any new handwriting of eight that it has not seen, it should be able to predict the correct number.

For here onwards, we will focus on deep learning only.

### Mathematical Definition

The main idea is, a neural network tries to approximate a function. Given a dataset of inputs and outputs, we assume that there was some underlying function that produced the output values given the input values and its the goal of a neural network to approximate this function as close as possible.

Given some function $$ y = f^*(x) $$, a neural network defines a mapping $$ y = f(x) $$ which is a close approximation to $$ f^* $$.

The branch of mathematics related to this is called Function approximation. In the mathematical theory of artificial neural networks, there is a theorem called Universal approximation theorem, which states that

> A neural network comprising of an output layer and at least one hidden layer with an activation function can approximate any function.

An important point to note here is that, neural networks are universal approximators, ie, they can be used to approximate any kind of function.

So, now we know, what is machine learning and what problem does deep learning solve. With the basics understood, let us now move ahead with understanding forward propogation.