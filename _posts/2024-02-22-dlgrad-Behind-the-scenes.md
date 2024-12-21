---
layout: post
title: "dlgrad: Behind the scenes"
date: 2024-02-22
markdown: kramdown
---

##  History

<details>
<summary> A brief timeline of dlgrad </summary>

<ul>
<li> I started this project in 2022 with the intention of learning the fundamentals of deep learning. The initial version worked perfectly fine but was just a numpy wrapper.</li>
<li> In early 2024, I revisted the project and realised that I didnt learn or do much since most of the heavy lifting was done by numpy and this bothered me.</li>
<li> Hence, I began to rewrite dlgrad, well, in a stupid way. </li>
<li> Since, I didnt want to rely on numpy at all, I needed some way of creating the tensors. My genius idea was, let me write C code in python, compile them as a shared file (using subprocess) and load them into python. Suprisingly it worked. The rational was, I wanted *dlgrad* to be a simple pip install, and didnt want to deal with compiling C code.</li>
<li> However, it was becoming really difficult to manage tensors in C and using them in python. Things were only getting complicated as I sarted to add new ops, losses, etc. And I spent around 8 months doing this. Yea 8 months !!!</li>
<li> At this point I became frustated at myself, saddend by the fact that I am not able to do this.</li>
<li> Then I was looking at [llm.c](https://github.com/karpathy/llm.c), and I wondered, why am I complicating things. All this complexity was arising from the fact that I didnt want to compile C code when installing. But, by doing that, I will drasctically improve performance, increase speed and reduce complexity.</li>
<li> I am not worried about the time since, as Andrej Karpathy mentions in the Lex podcast, these are just scar tissues. I have learnt from the mistake and hopefully will not repeat it in the future :). Hence, the lesson learnt here is that,      
    <ul>
        <li> <b> Don't complicate things. </b> </li>
        <li> <b> Before starting out on a project, layout a plan, figure out how you are going to do things beforehand, so that in the future, after putting so much effort on something, it should not come to a hault, because, you didnt think it through enough. </b> </li>
    </ul>
</li>
</ul>
</details>


## dlgrad Stack

(insert fig)

- For the CPU backend, it uses C for creating the Tensors and performing all the ops. Uses [cffi](https://cffi.readthedocs.io/en/stable/) to compile all the C source code on the user's machine when installing through pip.

## Broadcasting

dlgrad also supports broadcasting. It does <i><b>not</b></i> create any additional copies or take extra memory for this operation. Since all the Tensors irrespective of their shapes, are stored as 1D arrays, it took me a couple of days to figure out the algorithm, which can be read in the ops [c src files](https://github.com/NavneetKanna/dlgrad/tree/main/dlgrad/src/c), such as add or matmul.


## Explanation of algorithims

### Sum

I have created separate functions for each dimension of the tensor and for each axis, not sure if this is efficient but it does the trick. It was easier for me to figure the algorithm for each case than finding a generic one which applies for all cases. The source code can be found [here](https://github.com/NavneetKanna/dlgrad/blob/main/dlgrad/src/c/sum.c).

3D Tensor

Lets assumse a Tensor of shape (4, 3, 2)

```c
// The numbers show the indexes of the Tensor

[[[ 0,  1],
  [ 2,  3],
  [ 4,  5]],

 [[ 6,  7],
  [ 8,  9],
  [10, 11]],

 [[12, 13],
  [14, 15],
  [16, 17]],

 [[18, 19],
  [20, 21],
  [22, 23]]]

// shape = (4, 3, 2)
// stride = (6, 2, 1)
```

case sum(dim=0)
```c
// This is the main function 

for (int i=0; i<shape[1]; i++) { // rows
    for (int j=0; j<shape[2]; j++) { // cols
        float sum = 0.0;
        for (int k=i*strides[1]+j; k<numel*shape[0]; k+=strides[0]) {
            sum += arr[k];
        }
        out[idx] = sum;
        idx += 1;
    }
}

// Things will be easier if we print out the indexes on every looop

/*
    i=0 j=0 k=0
    i=0 j=0 k=6
    i=0 j=0 k=12
    i=0 j=0 k=18
    i=0 j=1 k=1
    i=0 j=1 k=7
    i=0 j=1 k=13
    i=0 j=1 k=19
    i=1 j=0 k=2
    i=1 j=0 k=8
    i=1 j=0 k=14
    i=1 j=0 k=20
    i=1 j=1 k=3
    i=1 j=1 k=9
    i=1 j=1 k=15
    i=1 j=1 k=21
    i=2 j=0 k=4
    i=2 j=0 k=10
    i=2 j=0 k=16
    i=2 j=0 k=22
    i=2 j=1 k=5
    i=2 j=1 k=11
    i=2 j=1 k=17
    i=2 j=1 k=23
*/

// The main thing to look at is the innermost loop k, it is pretty self-explanatory what the algorithm is doing 
// by comparing the k values with the array shown above.
```

case sum(dim=1)
```c
// This is the main function 

float *sum_3d_dim1(float *arr, int numel, int *shape, int *strides) {
    float *out = malloc(sizeof(float)*numel);
    int idx = 0;
    for(int i=0; i<shape[0]*strides[0]; i+=strides[0]) {
        for (int j=0; j<shape[2]; j++) { // cols
            float sum = 0.0;
            for(int k=i+j; k<(i+strides[0]); k+=strides[1]) { // rows
                sum += arr[k];
            }
            out[idx] = sum;
            idx += 1;
        }
    }

    return out;
}

/*
    i=0 j=0 k=0
    i=0 j=0 k=2
    i=0 j=0 k=4
    i=0 j=1 k=1
    i=0 j=1 k=3
    i=0 j=1 k=5
    i=6 j=0 k=6
    i=6 j=0 k=8
    i=6 j=0 k=10
    i=6 j=1 k=7
    i=6 j=1 k=9
    i=6 j=1 k=11
    i=12 j=0 k=12
    i=12 j=0 k=14
    i=12 j=0 k=16
    i=12 j=1 k=13
    i=12 j=1 k=15
    i=12 j=1 k=17
    i=18 j=0 k=18
    i=18 j=0 k=20
    i=18 j=0 k=22
    i=18 j=1 k=19
    i=18 j=1 k=21
    i=18 j=1 k=23
*/

// Again looking at the innermost loop k, it is pretty self-explanatory what the algorithm is doing 
// by comparing the k values with the array shown above.
```