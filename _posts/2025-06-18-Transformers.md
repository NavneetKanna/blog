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

inp = (2, 6, 4)

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
        [0.4553, 0.3277, 0.4210, 0.6628],   # The
        [0.0874, 0.3216, 0.2850, 0.1438],   # sun
        [0.8880, 0.2221, 0.6271, 0.8234],   # dipped
        [0.9180, 0.8070, 0.4281, 0.1977],   # below
        [0.4874, 0.9018, 0.4258, 0.1630],   # the
        [0.0441, 0.1988, 0.6751, 0.8757],   # horizon
    ],
    [
        [0.9347, 0.9255, 0.6341, 0.0567],   # ,
        [0.7656, 0.2911, 0.6161, 0.0123],   # painting
        [0.4200, 0.3295, 0.1863, 0.7694],   # the
        [0.3197, 0.9724, 0.6066, 0.2184],   # sky
        [0.4521, 0.9276, 0.3951, 0.1281],   # with
        [0.7899, 0.7894, 0.2245, 0.2889]    # hues
    ]
]

"""
We have 2 batches, each with 6 rows (these are words from the dataset, since that is our context length)
and 3 columns (the length of our embedding dimension).
"""
```

This input now gets fed into individual blocks which are independent of each other.

### Block

The block consists of multiple attention heads, layer normalization layers, and a feed-forward layer.

1. The input is first passed through a layer normalization layer. (In the original paper, layer normalization is applied after the attention heads, but recently, it has become more common to apply it before the attention heads.)
2. The normalized input is then fed into the individual attention heads.
3. The outputs from the multi-head attention are added to the original input.
4. The result of this addition is passed through another layer normalization layer.
5. The normalized output is then fed into the feed-forward layer.
6. Finally, the output of the feed-forward layer is added to the output from step 3.

#### Multi-Attention heads

1. The input is fed into multiple attention heads in parallel.
2. The outputs from each attention head are concatenated.
3. The concatenated output is then passed through a linear layer.

#### Single Attention head (Attention mechanism)

First lets understand what are the goals of attention mechanism, what does it try to achieve,

1. Decide what parts of the input sequence (the context length) to focus on when processing or generating each token.

For our example, lets take the first input sequence, ```The sun dipped below the horizon``` (6 is our context length). Let's say we are predicting the last word ```The sun dipped below the ____```, to predict this word, it needs to learn that the word ```sun``` or ```dipped``` is more important or relevant compared to others, and give more attention towards those words.

To accomplish this, there are 3 vectors that are used, query, key and value. The intuition is as follows

| Name          | Intuition                 |
| ------------- | -------------------------- |
| **Query (Q)** | What do I need?            |
| **Key (K)**   | What do I have?            |
| **Value (V)** | What info can I provide?   |

Each token in the sequence has got all 3 vectors associated with them. The way they are derived is by shifting them from embedding space into a query, key and value space using a linear transformation ```nn.linear(bias=False)```. The weights associated with this linear layer are learnt during training. In other words, the model tries to learn a good weight matrix that can transform the input embedding into reasonable representations of the query, key and values space for the given dataset.

Ok now lets try to understand what these are actually.

Given the input, in our case, (2, 6, 4), we get the query, key and value matrix like so

```python
self.key = nn.Linear(n_embd, head_size, bias=False)
self.query = nn.Linear(n_embd, head_size, bias=False)
self.value = nn.Linear(n_embd, head_size, bias=False)
```

where ```n_embd``` is 4. ```head_size``` is calculated as follows

```python
head_size = n_embd // n_head
```

```n_head``` is a parameter we can choose based on the embedding dimension, for example, if the embedding dimension is 4, we can set ```n_head``` to 2, so that ```head_size``` is 2 which means each head processes 2 columns from the input.
