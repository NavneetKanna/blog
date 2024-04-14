---
layout: post
title: "dlgrad: Behind the scenes"
date: 2024-02-22
---

dlgrad is an autograd engine with PyTorch like api that I had built about an year ago (2022), insipired by andreaj karapthy's micrograd and geogre hotz's tinygrad. With dlgrad, you can train MLP's and CNN's, which I was proud to tell. However, upon closer inspection (in 2023), I realized that the design of the project was flawed, and since I didn't have access to a GPU back then, it was designed solely for CPU usage. Additionally, even though I had implemented all the algorithms from scratch, the entire project relied on NumPy for everything.

Now, after purchasing an M2 Air with access to a GPU and gaining an intermediate understanding of [Metal Shading Language (MSL)](/_posts/2024-01-30-Understanding-Metal-and-MSL.md), I am ready to rewrite this beautiful project from the ground up. This time, I aim to eliminate the usage of NumPy, implement support for Metal (and CUDA in the future), and improve the overall structure of the project.

So, lets begin!

## Architecture

It is pretty evident that if I have to eliminate the use of numpy, I have to create data buffers (contiguous memory) myself, there are two ways of doing this as far as my knowledge goes:

1. Use Python's *array* module.
2. Use *ctypes* to read shared objects of C files.

While the former can be used to create memory buffers, the biggest drawback is that I cannot perform matrix multiplication between arrays without the use of NumPy, ruling it out. Therefore, making the second approach the only option, which is good for me, because, I get to code in C (which I enjoy) and hence improve my C skills.

Intially, I started with C source code files and compiling them to shared libraries. But, I soon realised, if I do it this way then people have to compile the C source code when installing dlgrad. I dont like this, because I believe that a good project should have almost 0 dependencies, and I want dlgrad to be as simple as a pip install. 

So the way I am going to do this is by writing C code inside python, compiling it using clang through subprocess, and then read the shared object using ctypes. This shared object will be present throughout the lifetime of the program and will be automatically deleted once the program ends gracefully.

There are two kinds of ops dlgrad supports:
1. **BufferOps** create data buffers.
2. **ElementwiseOps** are UnaryOps, BinaryOps.
3. **MatrixOps** perform matrix calculations such as matmul's.

The below diagram shows the call grapgh of how data buffers are created,

![bufferops]({{ "/assets/images/bufferops.svg" | prepend: site.baseurl }})

