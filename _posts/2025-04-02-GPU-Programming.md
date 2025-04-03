---
layout: post
title: "GPU Programming"
date: 2025-04-02
---

I'm documenting my GPU programming journey through this blog post. I do not have nvidia gpu's, but I do however have apple m2. The concepts are similar irresepective of the manufacturer.

### Grid, Threadblocks, Warps

- Given a grid, which can be 1D, 2D or 3D, for example, a matrix or an image, it is subdivided into (thread)blocks of size n, which is further subdivided into groups of 32. 
- We can think of a 2D matrix, lets say 4096x4096, as a grid of 4096x4096 threads. This grid is divided into blocks of size n, lets say 1024 (32 width-wise, 32-height wise, 1 depth-wise).
  The 32x32x1 blocks are further divided into 1D group of size 32 called warps. So in this case, there are 32 warps for this block.
  
  