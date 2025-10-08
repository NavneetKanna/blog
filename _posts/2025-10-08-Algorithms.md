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
A = [[0, 1, 2
      3, 4, 5]]

(3, 4)
B = [[0, 1, 2, 3
      4, 5, 6, 7
      8, 9, 10, 11]]

(2, 4)
C = [[C(0, 0), C(0, 1), C(0, 2), C(0, 3)
      C(1, 0), C(1, 1), C(1, 2), C(1, 3)]]

To compute the first element of C(0, 0) we have to multiple and add the first row of A and first column of B.

C(0, 0) = A(0, 0)*B(0, 0) + A(0, 1)*B(1, 0) + A(0, 2)*B(2, 0)
C(0, 0) = 0*1 + 1*5 + 2*9
C(0, 0) = 23

To compute the second element of C(0, 1) we have to multiple and add the first row of A and second column of B.

C(0, 1) = A(0, 0)*B(0, 1) + A(0, 1)*B(1, 1) + A(0, 2)*B(2, 1)
C(0, 1) = 0*2 + 1*6 + 2*10
C(0, 1) = 26

To compute the second element of C(1, 0) we have to multiple and add the second row of A and first column of B.

C(1, 0) = A(1, 0)*B(0, 0) + A(1, 1)*B(1, 0) + A(1, 2)*B(2, 0)
C(1, 0) = 3*1 + 4*5 + 5*9
C(1, 0) = 68

and so on

```

### CPU implementation

```c
int rowsA = 2;
int colsB = 4;
int colsA = 3;
for (int rowA = 0; rowA < rowsA; rowA++) {          // Iterate through rows of Matrix A or Matrix C
    for (int colB = 0; colB < colsB; colB++) {      // Iterate through columns of Matrix B or Matrix C
        out[rowA * colsB + colB] = 0;               // Initialize element of result matrix

        for (int sharedIndex = 0; sharedIndex < colsA; sharedIndex++) { 
            out[rowA * colsB + colB] += A[rowA * colsA + sharedIndex] * B[sharedIndex * colsB + colB];
        }
    }
}

```

To make things simpler, we can print out the indices

```
out[0] += A[0] * B[0]
out[0] += A[1] * B[4]
out[0] += A[2] * B[8]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[1] += A[0] * B[1]
out[1] += A[1] * B[5]
out[1] += A[2] * B[9]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[2] += A[0] * B[2]
out[2] += A[1] * B[6]
out[2] += A[2] * B[10]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[3] += A[0] * B[3]
out[3] += A[1] * B[7]
out[3] += A[2] * B[11]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[4] += A[3] * B[0]
out[4] += A[4] * B[4]
out[4] += A[5] * B[8]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[5] += A[3] * B[1]
out[5] += A[4] * B[5]
out[5] += A[5] * B[9]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[6] += A[3] * B[2]
out[6] += A[4] * B[6]
out[6] += A[5] * B[10]

--- innermost loop done (3 cols of A or 3 rows of B) ---

out[7] += A[3] * B[3]
out[7] += A[4] * B[7]
out[7] += A[5] * B[11]

--- innermost loop done (3 cols of A or 3 rows of B) ---
```

To understand the way the indexing works, we need to lay out the matrix the way it is laid out in memory

```

(2, 3)
a = [(0, 0), (0, 1), (0, 2)
     (1, 0), (1, 1), (1, 2)]
a = [0, 1, 2
     3, 4, 5]

In memory, eveything is laid out as 1D.

[0, 1, 2, 3, 4, 5]

Now to access say (1, 2) element, the formula is

a(1, 2) = current_row * no_of_cols + current_column

When we use for loops to traverse through the rows and columns, this multiplication (current_row * no_of_cols) takes the pointer to the starting index of a row, in this case 3. Next the additon (+ current_column) offsets the pointer to the column required, in this case 2. Therefore 

a(1, 2) = (1*3) + 2
a(1, 2) = 5

