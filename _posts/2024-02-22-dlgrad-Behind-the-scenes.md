---
layout: post
title: "dlgrad: Behind the scenes"
date: 2024-02-22
---

dlgrad is an autograd engine with PyTorch like api that I had built about an year ago from scratch insipired by andreaj karapthy's micrograd and geogre hotz's tinygrad. Using dlgrad, you can train MLP's and CNN's, which I was proud to tell. But looking at it now, I notice that the way it is designed is bad, and since I had no access to a GPU back then it was designed only for CPU's and even though I had implemented all algorithms from beginning, the whole project was utilising numpy for everything. 

But now after I have purchased an M2 air and access to a GPU and having an intermediary understanding of [MSL](/_posts/2024-01-30-Understanding-Metal-and-MSL.md) and having matured a little bit more (than compared to a year ago), I am ready to re-write this beautiful project again but this time reducing the usage of numpy as much as possible, having an metal support, and an overall better structure. So, lets begin :).