---
layout: post
title: "Part 1:  What is machine learning ?"
date: 2023-12-25
---

Before delving into the realms of machine learning, it's essential to understand its fundamental aspects: What is it? What problems does it aim to solve? And why does it work?

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

Given sufficient data for training, ML algorithms can learn the underlying patterns and be able to generalise to new data. So for example, the algorithm learns the pattern that if there are two curves on top of each other, then it is most likely an eighth, and so on. And now given any new handwriting of eighth that it has not seen, it should be able to predict the correct number.
