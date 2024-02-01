---
layout: post
title: "Understanding Metal and MSL"
date: 2024-01-30
---

- Metal is an API/framework provided by apple to interact with the GPU; MSL is a language used to write kernels using the features provided by metal.
- MSL is C++ 14 based.

## Steps to execute a kernel function 

1. Write the main compute kernel function in a file ending with *.metal*
2. Get instance of the GPU to communicate with it, this is done using *MTLCreateSystemDefaultDevice()*
```
device = MTLCreateSystemDefaultDevice()
```
3. Get an instance of the metal library that contains the compute function we want to run. This can be done in two ways:
    - Write the kernel code inside of a docstring and call *device.newLibraryWithSource_options_error_(prg, ..)*