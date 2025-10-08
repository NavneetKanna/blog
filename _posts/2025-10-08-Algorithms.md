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

The same thing applies to the matmul code. A[rowA * colsA + sharedIndex], here sharedIndex is nothing but colA. So we are accessing A[0], A[1] and so on. For B B[sharedIndex * colsB + colB], it is slightly different, since in the inner most loop we are traversing along the colsA or rowsB, for every row of A we want to get the corresponding col of B.
```

Now the above implementation is not cache-friendly. In other words, it is an efficient algorithm. It makes sense, since if we look at B matrix, for every iteration of the loop we are skipping colsB/sharedIndex length to retrive the next item. This is not good.

Whats good for the cache and speed is that we retrieve/store elements that are adjacent to one another.

If we go back to the way matmul is done

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

C(0, 0) = A(0, 0)*B(0, 0) + A(0, 1)*B(1, 0) + A(0, 2)*B(2, 0)
C(0, 1) = A(0, 0)*B(0, 1) + A(0, 1)*B(1, 1) + A(0, 2)*B(2, 1)
...

Splitting it up

C(0, 0) = A(0, 0)*B(0, 0) + ... + ...
C(0, 1) = A(0, 0)*B(0, 1) + ... + ...
C(0, 2) = A(0, 0)*B(0, 2) + ... + ...
C(0, 3) = A(0, 0)*B(0, 3) + ... + ...

C(0, 0) = ... + A(0, 1)*B(1, 0) + ...
C(0, 1) = ... + A(0, 1)*B(1, 1) + ...
C(0, 2) = ... + A(0, 1)*B(1, 2) + ...
C(0, 3) = ... + A(0, 1)*B(1, 3) + ...

C(0, 0) = ... + ... + A(0, 2)*B(2, 0)
C(0, 1) = ... + ... + A(0, 2)*B(2, 1)
C(0, 2) = ... + ... + A(0, 2)*B(2, 2)
C(0, 3) = ... + ... + A(0, 2)*B(2, 3)
```

We can observe that for the for every element in a row of C, adjacent elements of A and B are used. Or in other words, for every row of C, if we loop through it colsB times and keep adding the respective element, we have essentially done matmul. We do not have to skip colsB time everytime. 

In the C code, all we have to do to achieve this, is by interchanging the last and second last loop

```c
int rowsA = 2;
int colsB = 4;
int colsA = 3;
for (int rowA = 0; rowA < rowsA; rowA++) {                   // Iterate through rows of A or C
    for (int sharedIndex = 0; sharedIndex < colsA; sharedIndex++) {   // Iterate through shared dimension
        for (int colB = 0; colB < colsB; colB++) {           // Iterate through columns of B or C
            // Assuming 'out' is already zeroed before this loop:
            out[rowA * colsB + colB] += A[rowA * colsA + sharedIndex] * B[sharedIndex * colsB + colB];
        }
    }
}
```

```
out[0] += A[0] * B[0]
out[1] += A[0] * B[1]
out[2] += A[0] * B[2]
out[3] += A[0] * B[3]

--- innermost loop done (4 cols of B) ---

out[0] += A[1] * B[4]
out[1] += A[1] * B[5]
out[2] += A[1] * B[6]
out[3] += A[1] * B[7]

--- innermost loop done (4 cols of B) ---

out[0] += A[2] * B[8]
out[1] += A[2] * B[9]
out[2] += A[2] * B[10]
out[3] += A[2] * B[11]

--- innermost loop done (4 cols of B) ---

out[4] += A[3] * B[0]
out[5] += A[3] * B[1]
out[6] += A[3] * B[2]
out[7] += A[3] * B[3]

--- innermost loop done (4 cols of B) ---

out[4] += A[4] * B[4]
out[5] += A[4] * B[5]
out[6] += A[4] * B[6]
out[7] += A[4] * B[7]

--- innermost loop done (4 cols of B) ---

out[4] += A[5] * B[8]
out[5] += A[5] * B[9]
out[6] += A[5] * B[10]
out[7] += A[5] * B[11]

--- innermost loop done (4 cols of B) ---
```

If looking at it from 2D perspective makes it easier

```c
// naive/not cache efficient method
for (int rowA = 0; rowA < n; rowA++) {
    for (int colB = 0; colB < n; colB++) {
        for (int sharedIndex = 0; sharedIndex < n; sharedIndex++) {
            out[rowA][colB] += A[rowA][sharedIndex] * B[sharedIndex][colB];
        }
    }
}


// cache efficient method
for (int rowA = 0; rowA < n; rowA++) {
    for (int sharedIndex = 0; sharedIndex < n; sharedIndex++) {
        for (int colB = 0; colB < n; colB++) {
            out[rowA][colB] += A[rowA][sharedIndex] * B[sharedIndex][colB];
        }
    }
}

```