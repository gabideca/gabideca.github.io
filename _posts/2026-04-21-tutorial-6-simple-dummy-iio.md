---
title: "Tutorial 6 - Anatomy of a dummy driver (IIO)"
date: 2026-04-21 21:08:22 -0300
categories: [Kernel, Tutorials]
tags: [linux, kernel, iio, drivers, dummy, sensors]
---

Moving on to the 6th tutorial, straight to it: IIO.

At last, it is time we delve deep into the subsystem itself.

You know the gist, FLUSP's [tutorial](https://flusp.ime.usp.br/iio/iio-dummy-anatomy/).

## Tutorial 6
This tutorial covers the anatomy of the `iio_simple_dummy`, present in the kernel with the express role of serving as a reference driver
for anyone interested. How nice. Thank you Linux kernel.

The tutorial goes over:
* IIO channels
* The `iio_chan_spec` struct inner workings
* Channel-separated and direction-shared attributes
* Event registering
* General initialization and reomval of an IIO driver.

All and all, as intimidating as the guide might seem at first, it actually went pretty smoothly.

## Observation
As might be clear by the list above, the `iio_chan_spec` is key for this tutorial.

The special note goes to its remarkable configrability, allowing it to represent everything from a voltage channel to an accelerometer's axis.
The user can easily manage the relevant attributes through the `info_mask_*` fields.

Another observation goes to the care put into the tutorial. This is not exclusive to this one in particular by any means, but it was thoroughly annotated which helped immensely.
