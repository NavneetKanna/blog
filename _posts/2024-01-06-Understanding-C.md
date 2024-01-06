---
layout: post
title: "Understanding C"
date: 2024-01-01
---


This blog is intended to understand:
1. c fundamentals 
2. How a program is structed in memeory and  executed
3. some bits of os (I will have another blog post dedicated to only os)

This will be a Q and A type of blog, where all the questions are what I have asked myself while doing a c project and the answer from research.

**What is strtok() and strtok_r() ?**

I was looking for a function to tokenize a string and came across these. 

strtok() is a function in string.h which outputs tokens from a string *s* based on a delimetter *delim*.

It should be called multiple times to get successive tokens.

1. In the first call *s* should not be null.
2. It searches for the first character which is not contained in *delim* and makes a pointer say *tmp* to point to it.
3. That character is the beginning of the token. The function then searches from that point on for the first character that is contained in *delim*.
4. Now it replaces the *delim* character with null charcter and a global pointer variable stores the next character. And returns the *tmp* pointer.
5. In subsequent calls, *s* should be null, which means it will continue from where it left off and repeats the steps 2-4.

For example, let *s = "As If"*

Steps 1-2: 

![step1-2]({{ "/assets/images/strtok_1.svg" | prepend: site.baseurl }})

Steps 3-4:

![step3-4]({{ "/assets/images/strtok_2.svg" | prepend: site.baseurl }})

The problem with strtok() which you might have guessed is the use of a global variable, making it "thread unsafe", which means that, when strtok() is running in one thread and for some reason pauses execution, and, another thread calls strtok(), the value of the global variable will change which affects the calling program on the first thread as well, which is not desired.