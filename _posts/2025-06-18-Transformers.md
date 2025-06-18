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