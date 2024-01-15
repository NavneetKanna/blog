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

*strtok()* is a function in string.h which outputs tokens from a string *s* based on a delimetter *delim*.

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

The problem with *strtok()* which you might have guessed is the use of a global variable, making it "thread unsafe", which means that, when *strtok()* is running in one thread and for some reason pauses execution, and, another thread calls *strtok()*, the value of the global variable will change which affects the calling program on the first thread as well, which is not desired.

The *strtok_r()* function is the thread-safe versrion of *strtok()* by having an additional argument called *state*, which is basically the global variable pointer (also called internal state) returned to user and should be passed as arguments in subsequent calls.

The *_r* in the function name is called reentrant. **Reentrancy** is a concept in computing which means that a function can be interrupted and safely be called by another program (re-entered) without any ill-effect.

Based on my explanation, I hope it is clear that *strtok()* is not a reentrant function and *strtok_r()* is one.

**Strings in c**

In c, anything inside double quotes is called a *string literal*. C stores these string literals as null-terminated char arrays. 

There are two ways to define a string:

1. Using a char array

```c
char s[] = "hello"
```

2. Using a pointer

```c
char *s = "hello"
```

For the first case, s is an array and a pointer in the second. However, there is one major difference:

1. In the array version, the chars can be modfied. But, in the pointer version, we cannot modify the string.

For a function expecting a char *, either an array or a pointer can be passed, since, the array automatically decays to a pointer to its first element.

**What is size_t and ssize_t**

1. Unsigned integer type that is used to represent the size of objects in memory. 
2. *size_t* are used as loop index because sometimes the index can be a larger value than the maximum value that can be represented by an int (2^32-1).
3. *ssize_t* is the signed version which accounts for -1 as error return type (-1, SSIZE_MAX).

***getchar()*: assigning the returned int value to a char arryay**

So it is possible to do this:

```c

```