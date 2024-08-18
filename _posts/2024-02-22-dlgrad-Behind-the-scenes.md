---
layout: post
title: "dlgrad: Behind the scenes"
date: 2024-02-22
---

dlgrad is an autograd engine with PyTorch like api that I had built about an year ago (2022), insipired by andreaj karapthy's micrograd and geogre hotz's tinygrad. With dlgrad, you can train MLP's and CNN's, which I was proud to tell. However, upon closer inspection (in 2023), I realized that the design of the project was flawed, and since I didn't have access to a GPU back then, it was designed solely for CPU usage. Additionally, even though I had implemented all the algorithms from scratch, the entire project relied on NumPy for everything.

Now, after purchasing a M2 Air with access to a GPU and gaining an intermediate understanding of [Metal Shading Language (MSL)](/_posts/2024-01-30-Understanding-Metal-and-MSL.md), I am ready to rewrite this beautiful project from the ground up. This time, I aim to eliminate the usage of NumPy, implement support for Metal (and CUDA in the future), and improve the overall structure of the project.

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


## Weight initialisation 

Pytorch uses [kaiming uniform](https://github.com/pytorch/pytorch/blob/37596769d8b42beba104e14d149cebe0dfd75d12/torch/nn/modules/linear.py#L109) to initialise the weights and hence dlgrad will use that too. Kaiming uniform forumla calculates the bound that is used to generate random uniform numbers within. The formula is,

$$ bound = \sqrt{3} \cdot std $$

where,

$$ std = \frac{gain}{\sqrt{fan\_in}} $$
$$ gain = \sqrt{\frac{2}{1 + a^2}} $$
$$ fan\_in = no \ of \ inputs \ coming \ into \ a \ neuron $$
$$ a = the \ negative \ slope \ of \ the \ rectifier $$

hence the full equation becomes,

$$ bound = \sqrt{3} \cdot \frac{\sqrt{\frac{2}{1 + a^2}}}{\sqrt{fan\_in}} $$

Now, we have to generate random uniform numbers in the range (-*bound*, *bound*). In torch nn.linear, *a* is given the value of $\sqrt{5}$, this [issue](https://github.com/pytorch/pytorch/issues/15314) explains why.

For example, considering the MNIST dataset, the input layer will have 784 neurons and say the first hidden layer has 64 neurons, hence, $fan\_in = 784$ and $fan\_out = 64$ and assuming $a = \sqrt{5}$, 

$$ bound = 1.732 * \frac{0.577}{28} $$
$$ bound = 0.035 $$

I have verified the correctness of the value by adding some `print` statements in the [`kaiming_uniform_()`](https://github.com/pytorch/pytorch/blob/470723faea17e05f22001b85c505d2acafc53902/torch/nn/init.py#L457) function. 

## xoroshiro

I encountered a significant roadblock, but fortunately, I've found a solution. After figuring out how to initialize weights (as discussed in the previous topic), I needed to modify the C function I was using to accept bounds. I had been using the rand() function in C to generate random numbers, but I knew this wasn't the best option, both from my own knowledge and from my research.

I began researching better or recommended ways to generate random uniform numbers in C. I discovered that PyTorch uses the Mersenne Twister generator. I wanted to implement it, but I couldn't find a good reference or pseudocode, and the authors had copyrighted the code.

Then, I found the Xorshift generator, which has no copyright restrictions and is small and fast. I'm currently using [*xoroshiro64**](https://prng.di.unimi.it), a 32-bit generator (since dlgrad currently only supports fp32). You can find the C code [here](https://prng.di.unimi.it/xoroshiro64star.c).

To convert the output to a float in the range (0, 1), I divide by UINT32_MAX, because the generator produces numbers in the range (0, UINT32_MAX). Finally, to convert the random number to a range (-bound, bound), I use the formula provided [here](https://stackoverflow.com/questions/5294955/how-to-scale-down-a-range-of-numbers-with-a-known-min-and-max-value).

## Broadcasting

The way broadcasting works in dlgrad is that, there are no new tensors (c arrays) created, but rather the way the array is referenced when performing different operations. For example, let us consider addition,

```c
// (3, 3)
a = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

```
