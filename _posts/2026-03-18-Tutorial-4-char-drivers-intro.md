---
title: "Tutorial 4 - Character Device Drivers Intro"
date: 2026-03-18 22:22:11 -0300
categories: [Kernel, Tutorials]
tags: [linux, kernel, drivers, char-device, file-operations]
---

No dilly-dallying this time around. All the contextual information needed has been provided in the previous posts, this time we can jump straight into the action!

This tutorial covers Character Devices, arguably one of the most powerful way to exchange information between drivers and the user space.
You're up to speed by now, you know where to find the [tutorial](https://flusp.ime.usp.br/kernel/char-drivers-intro/)

## What is Covered
* Character devices
* Major and minor numbers
* File operations

## The Caveat
If the reader knows a thing or two about kernel (unlike I did), the aforementioned list should have rang some bells - 
this tutorial is a considerable step-up in difficulty in relation to the previous ones.
That doesn't mean this post will have a giant issues section, but it should be registered that this tutorial took more time to finish than the previous ones.

## Issue 1 - identifying script roles
While following this tutorial, a frequent issue was not knowing what the role of the current script I was editing was, specifically.
For example, while trying to get the `simple_char` driver to print out its major and minor number, I couldn't figre out which documents I had to edit.

## Solution 1 - Asking for help
Yeah, it wasn't an all-nighter or some miracle eureka moment. I asked for help from a TA and he helped me figure out what the issue was.
Turns out, this time I wasn't at fault as all the steps I performed were correct and ordered. 
To be honest, I regret to say I am not sure what was wrong but I do believe the process of trying to finding it out helped me a lot to figuring out
how kw works, especially the `kw --send` command.

In the end, we got a simple character device up and running in the VM capable of reading and writing.
