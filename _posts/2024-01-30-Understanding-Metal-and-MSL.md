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
```python
device = Metal.MTLCreateSystemDefaultDevice()
```
3. Get an instance of the metal library that contains the compute function we want to run. This can be done in two ways:
    - Write the kernel code inside of a docstring and call *device.newLibraryWithSource_options_error_(prg, ..)*
    ```python
    options = Metal.MTLCompileOptions.new()
    lib = device.newLibraryWithSource_options_error_(prg, options, None)
    func_name = lib[0].newFunctionWithName_("addition_compute_function")
    ```
    - Or, write the kernel code in a seperate file with extension *.metal* and [compile it](https://developer.apple.com/documentation/metal/shader_libraries/building_a_shader_library_by_precompiling_source_files?language=objc)
    ```bash
    xcrun -sdk macosx metal -o test.ir  -c test.metal
    xcrun -sdk macosx metallib -o test.metallib test.ir
    ```
    and then call *newLibraryWithURL_error_("test.metallib)*
    ```python
    lib = device.newLibraryWithURL_error_("test.metallib", None)
    func_name = lib[0].newFunctionWithName_("addition_compute_function")
    ```
    if we print *func_name* we should see the compute kernel function name, device, function type and attributes (maybe the things displayed might vary based on the version of )

