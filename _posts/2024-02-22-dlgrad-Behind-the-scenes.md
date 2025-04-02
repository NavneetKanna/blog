---
layout: post
title: "dlgrad: Behind the scenes"
date: 2024-02-22
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

dlgrad also supports broadcasting. It does <i><b>not</b></i> create any additional copies or take extra memory for this operation. Since all the Tensors irrespective of their shapes, are stored as 1D arrays, it took me quite some time to figure out the algorithm. The code can be found [here](https://github.com/NavneetKanna/dlgrad/tree/main/dlgrad/src/c).


## Explanation of algorithims

### Binary ops

For 2D, the C function is

```c
// Handles all broadcasting shapes
void op_2d(float *x, float *y, float *out, int *xshape, int *xstrides, int *yshape, int *ystrides, int op)
{
    int x_idx = 0;
    int y_idx = 0;
    int y_idx2 = 0;
    int y_idx1 = 0;

    for (int i=0; i<xshape[0]; i++) {
        y_idx1 = (xshape[0] == yshape[0]) ? i : 0;
        for (int j=0; j<xshape[1]; j++) {
            y_idx2 = (xshape[1] == yshape[1]) ? j : 0;

            x_idx = i*xstrides[0] + j*xstrides[1];
            
            y_idx = y_idx1*ystrides[0] + y_idx2*ystrides[1];
            switch (op) {
            case ADD:
                out[x_idx] = x[x_idx] + y[y_idx];
                break;
            case MUL:
                out[x_idx] = x[x_idx] * y[y_idx];
                break;
            case SUB:
                out[x_idx] = x[x_idx] - y[y_idx];
                break;
            }
        }
    }
}
```

Let's consider we are adding 2 tensors of shape (2, 3) and (1, 3). The values are

```python
x.shape (2, 3)
x.stride (3, 1)
y.shape (1, 3)
y.stride (3, 1)
```

An important point to note is that, all the algorithms I have come up with assumes that the x array is the *higher* dimensional array or in other words the tensor to which the other tensor *should be broadcasted to*.

Now, the algorithm becomes

```c
 for (int i=0; i<2; i++) {          // rows
    for (int j=0; j<3; j++) {       // cols
    
    }
 }
```

The y_idx is simple *i* or *j* based on which dimension broadcasting has to be done multiplied by the stride.

```c

// If broadcasting has to be done along dimension 1 (row-wise), for example, (2, 3) and (1, 3), then y_idx1 is always 0 and y_idx2 is (0, 1, 2). 
// And y_idx is y_idx2*stride[1].

// If broadcasting has to be done along dimension 0 (col-wise), for example, (2, 3) and (2, 1), then y_idx1 is always (0, 1) and y_idx2 is always 0. 
// And y_idx is y_idx1*stride[0].

// x_idx is always row*xstrides[0] + col*xstrides[1]
```

The same logic applies to 3D tensors as well.

### Sum

For 2D, the C function is 

```c
void sum_2d(float *x, float *out, int *xshape, int *xstride, int outnumel, int dim)
{
    int out_idx = 0;
    int x_idx = 0;
    
    int nrows = xshape[0];
    int ncols = xshape[1];
    
    int row_stride = xstride[0];
    int col_stride = xstride[1];
    
    for (int row=0; row<nrows; row++) {
        for (int col=0; col<ncols; col++) {
            x_idx = row*row_stride + col*col_stride;
            switch (dim) {
                case 0:
                    out_idx = col;
                    out[out_idx] += x[x_idx];
                    break;
                case 1:
                    out_idx = row;
                    out[out_idx] += x[x_idx];
                    break;
            }
        }
    }
}
```

The algorithm is pretty simple, 

```c
// If we have to sum along dimension 0, then out_idx = col (0, 1, 2), and for the second row, it is again (0, 1, 2) and we increment the value out[out_idx].
// Hence we sum along rows.

// If we have to sum along dimension 1, then out_idx = row (0, 1), and for every col, we increment the value out[out_idx]. 
// Hence we sum along cols.
```

The same logic applies to 3D tensors as well.

### Max

The algorithm is the same as sum, but instead of incrementing *out[out_idx]*, we check if it less than *x[x_idx]* and update *out[out_idx]* as *out[out_idx] = x[x_idx]*  
