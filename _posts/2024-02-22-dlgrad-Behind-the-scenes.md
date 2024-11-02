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

dlgrad also supports broadcasting. It does <i><b>not</b></i> create any additional copies or take extra memory for this operation. Since all the Tensors irrespective of their shapes, are stored as 1D arrays, it took me a couple of days to figure out the algorithm, which is shown below

```c
// for 3D Tensor
int get_y_idx(int dim1, int dim2, int dim3, int *yshape, int *ystride) {
    int idx = 0;

    if (yshape[0] != 1) {
        idx += dim1*ystride[0];
    } 

    if (yshape[1] != 1) {
        idx += dim2*ystride[1];
    }

    if (yshape[2] != 1) {
        idx += dim3*ystride[2];
    }

    return idx;
}
```

Right now the *dlgrad* supports upto 4D, and the algorithm is slightly different for each dimension, but I will try to see if this can be done as one generic function in the future.

Now lets say we have two Tensor's x and y. Let x be of shape (4, 3, 2) and y be the *smaller* Tensor, that should be broadcasted.

Case 1:

y is of shape (3, 1) and stride (1, 1)

- First, the shape gets [padded with 1's to match the dimensions](https://data-apis.org/array-api/latest/API_specification/broadcasting.html), hence y shape and stride are now (1, 3, 1) and (1, 1, 1).
- Now, in the main function, we have 3 loop's going through each dimension, you can think of them as pointers.

```c
// for 3D Tensor
for (int i=0; i<xshape[0]; i++) {
    for (int j=0; j<xshape[1]; j++) {
        for (int k=0; k<xshape[2]; k++) {
            get_y_idx(i, j, k, yshape, ystride);
        }
    }
}
```

- Here's a detailed explanation,

```c
for (int i=0; i<4; i++) {
    for (int j=0; j<3; j++) {
        for (int k=0; k<2; k++) {
            get_y_idx(i, j, k, yshape, ystride);
        }
    }
}

// for (0, 0, 0) -> get_y_idx() will return 0
// for (0, 0, 1) -> get_y_idx() will return 0
// for (0, 1, 0) -> get_y_idx() will return 1, because yshape[1] != 1
// for (0, 1, 1) -> get_y_idx() will return 1, because yshape[1] != 1
// for (0, 2, 0) -> get_y_idx() will return 2, because yshape[1] != 1
// for (0, 2, 1) -> get_y_idx() will return 2, because yshape[1] != 1
// for (1, 0, 0) -> get_y_idx() will return 0
// for (1, 0, 1) -> get_y_idx() will return 0
// and so on
```

- To see it visually,

```
This is the x Tensor

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

and this is the y Tensor

[[0],
[1],
[2]]

Ideally after broadcasting, the y Tensor should look this

[[[ 0,  0],
 [ 1,  1],
 [ 2,  2]],

[[ 0,  0],
 [ 1,  1],
 [ 2,  2]],

[[ 0, 0],
 [ 1, 1],
 [ 2, 2]],

[[ 0, 0],
 [ 1, 1],
 [ 2, 2]]]

```

The same applies for the other 2 cases

- when y is of shape (3, 2)
- and when y is of shape (1, 2)

To summarise, the get_y_idx() will return the correct offset based on the stride, if the shape is not 1.