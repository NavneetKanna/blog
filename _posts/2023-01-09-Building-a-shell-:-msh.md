---
layout: post
title: "Building a shell: msh"
date: 2024-01-09
---

The idea behind this project is to better understand unix, os and c.

**M**inimal **Sh**ell, as I have called it since the goal is to implement important and useful commands. The idea of the structure of the program comes from [here](https://brennan.io/2015/01/16/write-a-shell-in-c/).

##First plan
One main loop running that will:
1. read input from stdin.
2. break it down to individual lexemes.
3. execute the command.