---
title: "Environment Setup - Handling Hardware Limitations and Linux Necessity"
date: 2026-02-27 19:37:45 -0300
categories: [Kernel, Tutoriais]
tags: [linux, arm64, qemu, libvirt, kernel]
---

This post marks the beggining of the Free Software Development course journey.
However, for reasons soon to be revealed, this post will not cover any free software topics just yet.

## The Issue - Previous Setup
For three years now, I had been meeting my Linux necessties through WSL - Windows Subsystem for Linux.
Up until this point, it had served me well and posed no issues whatsoever.

Unfortunately, in our very first class we were informed that students from previous years had attempted to finish the course in WSL and failed to do so.
As I felt like the course was enough of a challenge by itself, I chose to rule out WSL as a viable option.

Just as quick as WSL left the picture, so did a virtual machine (VM), as it was stated that those historically did not work as intended either.
Now, the two main ways I had been using Linux so far were eliminated. I had to find a way to run Linux natively.

## Attempt 1 - An Old Notebook
A first and obvious alternative would be creating a double boot out of my main notebook, but it didn't have the recommended storage for the course (indicated at around 512GB).
So I resorted to trying to reanimate an old notebook I had.

After a few days of tinkering and configuration, I managed to get it up and running again, but when I tried to
free the Linux partition Windows put a security file in the middle of it rendering the partition basically useless.

I tried defragmenting the disk, which by itslef took three days, and then a colleague introduced me a program whose biggest utility is to creating Windows partitions for Linux.
To no one's surprise, however, it was paid and the free version was of no use.

Eitherway, another colleague told me that if the notebook took as long as it did to turn on, the hard drive was probably as good as gone anyways and that running a Linux distro on it wouldn't make that much of a difference.

## Attempt 2 - The Last Resort
By this point, as probably noticeable, I had tried basically all I could easily achieve in a timely manner.

Therefore, I had to resort to asking to use a friends' terminal alongside him.

And it is at this point that I would like not only to introduce but to deeply thank André Jun Hirata, the man who saved this course.
I pitched him the idea and he started looking into it. In one week, he setup his own PC to be able to run two separate instances of qemu and made it so that I could follow the tutorials
you will be able to find in here as if I were in my own PC.

I invite you to check his personal blog and specific post regarding this process, in which he describes exactly how he managed to pull this off: [Post](https://andre-jun.github.io/posts/tutorial-1-qemu-libvirt-setup/)

## Final Product
At last, a genuine, mint-based, pristine Linux terminal all for me.
