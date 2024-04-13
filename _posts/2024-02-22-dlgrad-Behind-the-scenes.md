---
layout: post
title: "dlgrad: Behind the scenes"
date: 2024-02-22
---

dlgrad is an autograd engine with PyTorch like api that I had built about an year ago (2022) from scratch insipired by andreaj karapthy's micrograd and geogre hotz's tinygrad. Using dlgrad, you can train MLP's and CNN's, which I was proud to tell. But looking at it now (2023), I notice that the way it is designed is bad, and since I had no access to a GPU back then it was designed only for CPU's and even though I had implemented all algorithms from beginning, the whole project was utilising numpy for everything. 

But now, after I have purchased an M2 air with access to a GPU and having an intermediary understanding of [MSL](/_posts/2024-01-30-Understanding-Metal-and-MSL.md), I am ready to re-write this beautiful project again but this time eliminating the usage of numpy, having a METAL support (CUDA in future), and an overall better structure. 

So, lets begin :).