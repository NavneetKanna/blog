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

The below diagram shows the call graph of how data buffers are created,

![bufferops]({{ "/assets/images/bufferops.svg" | prepend: site.baseurl }})

This diagram shows the call graph for the other ops,

![otherops]({{ "/assets/images/otherops.svg" | prepend: site.baseurl }})

Summary:
- All data in dlgrad are C arrays.
- C code is complied and loaded dynamically from python (no separate C source files).
- Three ops supported: BufferOps, ElemenetwiseOps, MatrixOps.

## Indexing

This is an interesting problem if you create data buffers on your own. Because in C, data is created as contigous 1D array, whereas Tensors are usually n-dim. Hence, a function is needed that converts n-dim indexing to 1D indexing. One thing to note is that, dlgrad follows NCHW data format.

The code can be found [here](https://github.com/NavneetKanna/dlgrad/blob/60c40a06d0d4ce7ef372e6bf124e4e4b5506ef2a/dlgrad/tensor.py#L99). It is actually a very simple solution and beautifully written (proud of myself for this :D). The main part is this [function](https://github.com/NavneetKanna/dlgrad/blob/60c40a06d0d4ce7ef372e6bf124e4e4b5506ef2a/dlgrad/helpers.py#L39)

```python
def calculate_nchw_offset(n=0, c=0, h=0, w=0, N=0, C=0, H=0): 
    return (n * N) + (c * C) + (h * H) + w
```

Here, width (*w*) is not multiplied by anything because this dim moves the fastest, in other words, the elements are adjacent to each other.

Another important point to note from the code is that, any indexing of a Tensor is not a new Tensor (or a new data buffer), but rather just a view (different strides, offset, len, shape).

For example, lets assume a (2, 2, 2) Tensor

```python
a = [
    [[0.6095, 0.2799],
    [0.5239, 0.6648]],

    [[0.5869, 0.6684],
    [0.5006, 0.3380]]
]
```

This is how it will be represented in the memory as a 1D array and the dimensions

![offset]({{ "/assets/images/offset.svg" | prepend: site.baseurl }})

```python
# NOTE: strides = (4, 2, 1)

# Now lets say I want to access a[0]
# offset = 0*4
c = 0
offset = calculate_nchw_offset(c=c, C=strides[0]) 

# a[1, 1]
# offset = (1*4) + (1*2)
c, h = 1, 1 
offset = calculate_nchw_offset(c=c, h=h, C=strides[0], H=strides[1]) # 6
```