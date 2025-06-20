---
layout: post
title: "Transformers (Decoder-Only)"
date: 2025-06-18
---

In the post, I try to explain the transformer architecture (decoder-only) from scratch, so lets begin. 

### High level overview

1. The data is embedded along with position.
2. There are multiple blocks. Each block consists of multiple Multi-Head Attention layers, followed by layer norms, followed by feed-forward network.
3. After the blocks, there is a linear and softmax linear, which outputs probabilites.


### Input data

Lets come back to encoding the input later, but lets assume after embedding with position, the input is as follows,

```python

inp = (2, 4, 3)

"""
where,
    2 is the batch size
    4 is the context length
    3 is the embedding dimension
"""

"""
Let this be the dataset

The sun dipped below the horizon, painting the sky with hues of orange and pink.
A gentle breeze rustled the leaves, creating a soothing melody.
In that peaceful moment, the world seemed to pause and breathe.
"""

# Lets assume the input is as follows

inp = [
    [
        [0.4553, 0.3277, 0.4210],   # The
        [0.0874, 0.3216, 0.2850],   # sun
        [0.8880, 0.2221, 0.6271],   # dipped
        [0.9180, 0.8070, 0.4281]    # below
    ],
    [
        [0.4874, 0.9018, 0.4258],   # the
        [0.0441, 0.1988, 0.6751],   # horizon
        [0.9347, 0.9255, 0.6341],   # ,
        [0.7656, 0.2911, 0.6161]    # painting
    ]
]

"""
We have 2 batches, each with 4 rows (these are words from the dataset, since that is our context length)
and 3 columns (the length of our embedding dimension).
"""
```

This input now gets fed into individual blocks which are independent of each other.

### Block

The block comprises of multiple attention heads, layer norms and a feed-forward layer. 

1. The input first gets fed into a layernorm (in the original paper the layernorm comes after the attention heads, but in receent times, people are preferring to use it before the attention heads).
2. The output of that now gets fed into the individual heads.
3. The output of the multi-head attention gets added with the original input.
4. The output of the above step again gets fed into a layernorm.
5. The output of the above step now gets fed into the feedforward layer.
6. The output of the above step gets added with the output of step 3.

#### Multi-Attention heads

1. The input gets fed into multiple attention heads.
2. The outputs from each are concatenated.
3. The concatenated output is fed into a linear layer.

#### Single Attention head




