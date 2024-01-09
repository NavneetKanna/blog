---
layout: post
title: "Building a shell: msh"
date: 2024-01-09
---

The idea behind this project is to better understand unix, os and c.

**M**inimal **Sh**ell, as I have called it since the goal is to implement important and useful commands. The idea of the structure of the program comes from [here](https://brennan.io/2015/01/16/write-a-shell-in-c/).

So there will be one main loop running. This loop displays the prompt, reads the command, parses it and executes it.

This was my short-lived plan: use *fgets()* to read the input line and *strtok_r()* to tokensize it and store the tokens in an array. Now compared the first index with a symbol table and execute the command.

As I was implementating the second part, which is, parsing of the command with *strtok_r()*, I relaised that this is not an efficient solution, because, the original command string gets modified by *strtok_r()* and a copy of the token strings have to be made.