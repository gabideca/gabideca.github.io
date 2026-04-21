---
title: "Tutorial 3 - Introduction to Linux kernel build configuration and modules"
date: 2025-03-14 18:43:18 -0300
categories: [Kernel, Tutoriais]
tags: [linux, arm64, kbuild, kconfig, módulos, kernel]
---

Let's keep them coming!

Onto tutorial 3, where we leave the realm of compiling and environment setup and enter the wonderful world of modules and building.
Its finally time to start leaving our marks on the kernel - even if only locally for now

If you've been clicking the links in the previous posts, you might have realised all the tutorials covered so far
(and - spoiler alert - covered until the end of the course) are from FLUSP.

So, if you don't mind, I would like to briefly introduce the platform and group. FLUSP is the University of São Paulo's very own free software development group.
They have a [website](https://flusp.ime.usp.br/kernel/modules-intro/) which I highly recommend checking to find out more about them.
I guarantee you'll be surprised by the things they pull off and the quailty of the published material. I know I sure was :)

And guess what? This post's tutorial is in there as well, as always - [Tutorial 3](https://flusp.ime.usp.br/kernel/modules-intro/)

## Tutorial 3 - Prelude
Lets get rollin'

By which I mean, lets actually find out how this whole directory and VM stuff works.
This tutorial marks the point in which my Linux instance was finally ready and I could actually start doing the tutorials myself, so I had some catching up to do.

First and foremost, I had to get accustomed to the actual directory tree. 
As I had been following Guilherme's and André's progress, their `cd`s and `ls`s grew too fast for me to keep up.
So it was at this point I got to know especially the `IIO_TREE`, in which most of the short term work will be done.

Along with the file structure, I learned how to use SSH, how to connect to the host VM, the role of the `activate.sh` env script and especially how to work kw.
A major shoutout to Guilherme and André is now once again deserved, as they, more used to the whole setup by now, helped guide me through the tutorialand the workspace.

Thanks to them, steps like `menuconfig`, in which I got stuck, did not turn out to be an issue whatsoever.

Welp I said a lot and didn't cover a single tutorial 3 related topic, didn't I? Lets fix this.

## Module Building and Our First Real changes
In this tutorial, we created a module in `IIO`'s `drivers/misc` (and its relative `kconfig` and `makefile` entries),
installed it into the VM and checked the logs to see our very own messages in there!

For this tutorial, the only major difficulty I faced was actually sending the compiled scripts to the host VM.
The issue was that I got confused with the steps and was trying to send the `.c` scripts to the VM before compiling. 
I'm sure you can guess what the fix was. (compile it before. Duh.)
