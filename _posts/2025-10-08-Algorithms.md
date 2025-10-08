---
layout: post
title: "Algorithms (Deep Learning Ops)"
date: 2025-09-27
---


## Matrix Multiplication (Matmul)

$$C = A @ B$$

The last dim of A and the first dim of B should match. For example, (2, 4) = (2, 3) @ (3, 4).

It works by taking a row of A and a column of B and performing two ops between them, multiplication and addition.

```

(2, 3)
A = [[1, 2, 3
      4, 5, 6]]

(3, 4)
B = [[1, 2, 3, 4
      5, 6, 7, 8
      9, 10, 11, 12]]

(2, 4)
C = [[C(0, 0), C(0, 1), C(0, 2), C(0, 3)
      C(1, 0), C(1, 1), C(1, 2), C(1, 3)]]

To compute the first element of C(0, 0) we have to multiple and add the first row of A and first column of B.

C(0, 0) = A(0, 0)*B(0, 0) + A(0, 1)*B(1, 0) + A(0, 2)*B(2, 0)
C(0, 0) = 1*1 + 2*5 + 3*9
C(0, 0) = 38

To compute the second element of C(0, 1) we have to multiple and add the first row of A and second column of B.

C(0, 1) = A(0, 0)*B(0, 1) + A(0, 1)*B(1, 1) + A(0, 2)*B(2, 1)
C(0, 1) = 1*2 + 2*6 + 3*10
C(0, 1) = 44

To compute the second element of C(1, 0) we have to multiple and add the second row of A and first column of B.

C(1, 0) = A(1, 0)*B(0, 0) + A(1, 1)*B(1, 0) + A(1, 2)*B(2, 0)
C(1, 0) = 4*1 + 5*5 + 6*9
C(1, 0) = 83

and so on

```