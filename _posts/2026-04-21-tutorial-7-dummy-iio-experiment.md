---
title: "Tutorial 7 - Experiment with dummy (IIO)"
date: 2026-04-21 23:50:52 -0300
categories: [Kernel, Tutorials]
tags: [linux, kernel, iio, drivers, dummy, sensors]
---

Not much introduction needed for this one. It is the direct continuation of the previous tutorial, as it is dirty hands time. Lets get practical!

## Tutorial 7
(From FLUSP, you know it. [Tutorial](https://flusp.ime.usp.br/iio/experiment-one-iio-dummy/))

The idea for this tutorial is to get the module we got to know in tutorial 7, `iio_simple_dummy` up and running - and interact with it - inside the VM.

After completing the guide, we have:
* Enabled the `iio_dummy` driver via `.config`
* Compiled the driver's subdirectory
* Load the module with the good old `modrpobe` (and verifying with the "gooder" and "older" `lsmod` and `modinfo`)
* Mount `configs` and create a virtual dummy device

## Observations
As in the previosu tutorial, I dedicatea section to the most essential piece of the workspace for the guide.

This time, the honor goes  to `configs`, responsible for the creation and configuration of the kernel objects through userspace directory operations.
It was especially helpful for the instantiation of virtual devices in runtime, making it vital for testing.

Once again, smooth sailing with this one. Maybe I did understand how all of this kernel thing works afterall.
