---
layout: post
title: "Understanding Metal and MSL"
date: 2024-01-30
---

-Metal is an API/framework provided by apple to interact with the GPU; MSL is a language used to write kernels using the features provided by metal.
-MSL is C++ 14 based.

## Steps to execute a kernel function 

1. Write the main compute kernel function in a file ending with *.metal*