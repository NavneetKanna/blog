---
layout: post
title: "Hardware_1: Assembly Language"
date: 2024-02-22
---

This blog post is about me understanding assembly language, and hence the first in its series of understanding the underlying hardware.

I thought I would start learning with Arduino UNO R3 which has a 8 bit ATmega328P microcontroller. But I soon realiased that there are no debuggers available which is not good since for someone who wants to learning assembly its important to see the contents of registers, and not have to blink an led to check the output.

Hence, I have decided to start with the classic Intel 8086 microprocessor by using an online [8086 emulator](https://yjdoc2.github.io/8086-emulator-web/).

Before starting to learn assembly, lets understand few of the specs of 8086
1. 16 bit microprocessor.
2. Little Endian.
3. 16 bit data bus.
4. 8 general purpose registers (ax, bx, cx, dx, si, di, bp and sp), each of which is 16 bits wide.
5. Apart from general purpose registers there are segment (cs, ds, es, ss), special purpose (ip) ans flag registers.
6. There a total of 14 registers.

## Fundamentals

**Bits** a boolean value (1 or 0).
**Bytes** is 8 bits, ranges from (0-255) or (-128-127).
**Word** 2 bytes or 16 bits.
**DWORD** 32 bits or 4 bytes.
**QWORD** 64 bits or 8 bytes.
