---
layout: post
title: "Understanding Metal and MSL"
date: 2024-01-30
---

- Metal is an API/framework provided by apple to interact with the GPU; MSL is a language used to write kernels using the features provided by metal.
- MSL is C++ 14 based.

## Steps to execute a kernel function 

1. Get an instance of the GPU to communicate too, using *MTLCreateSystemDefaultDevice()*
```python
device = Metal.MTLCreateSystemDefaultDevice()
```
2. Get an instance of the metal library that contains the compute function we want to run. This can be done in two ways:
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
    Printing *func_name* should deisplay the compute kernel function name, device, function type and attributes (maybe the things displayed might vary based on the version of ...)
    Also, printing *lib* will show errors in the compute kernel code, if there are any. 
3. The compute kernel code is still not yet an executable code, to make it one, we have to create a pipeline. [*A pipeline specifies the steps that the GPU performs to complete a specific task*] (https://developer.apple.com/documentation/metal/performing_calculations_on_a_gpu?language=objc)
```python
func_pso = device.newComputePipelineStateWithFunction_error_(func_name, None)
```
A compute pipeline can run a single compute function, hence multiple compute pipelines are needed for multiple compute functions
4. Create a command queue, command queue is used to send work(command buffers) to the GPU
```python
q = device.newCommandQueue()
```
5. Create data buffers,  
```python
buff1 = device.newBufferWithLength_options_(int, Metal.MTLResourceStorageModeShared)
buff2 = ...
```
Right now the buffers are allocations of memory without a predefined format. *MTLResourceStorageModeShared()* indicates that both the CPU and GPU uses a shared memory
6. Create a command buffer, a command buffer holds sequence of encoded commands
```python
cmd_buf = q.commandBuffer()
```
7. Create an encoder
```python
encoder = cmd_buf.computeCommandEncoder()
```
8. Set the pipeline state and compute kernel function arguments data
```python
encoder.setComputePipelineState_(func_pso)
encoder.setBuffer_offset_atIndex_(buff1, 0, 0)
encoder.setBuffer_offset_atIndex_(buff2, 0, 1)
```
The second parameter is offest: an offset of 0 means the command will access the data from the beginning of a buffer. The third parameter is the index of the argument in the compute kernel function
9. Specify the grid size (thred count) and the thread group size
```python
grid_size = Metal.MTLSizeMake(arrayLength, 1, 1)
thread_group_size = Metal.MTLSizeMake(threadGroupSize, 1, 1)
```
10. Encode the command to dispatch the threads
```python
encoder.dispatchThreads_threadsPerThreadgroup_(grid_size, thread_group_size)
```
11. End the encoder, when there are no more commands to encode
```python
encoder.endEncoding()
```
12. Run the command buffer by commiting it to the queue
```python
cmd_buf.commit()
```
13. Optionally, the program can wait or do some other task while the GPU is running
```python
cmd_buf.waitUntilCompleted()
```