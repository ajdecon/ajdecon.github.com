---
layout: post
title: "Finding nvml.h"
date: 2012-11-06 01:21
comments: true
published: false
categories: hpc nvml nvidia gpu 
---

_File this entry under "preventing others from banging their head against a wall as long as I did"._

I've been playing with [NVIDIA's](http://www.nvidia.com/) [NVML](https://developer.nvidia.com/nvidia-management-library-nvml)
library recently. It provides a bunch of handy functions for querying the health and configuration of Tesla GPUs, setting compute
mode and ECC state, etc. 

However, while the [Python bindings](http://packages.python.org/nvidia-ml-py/) (and presumably the Perl ones) work just fine
accessing libnvidia-ml.so on their own, I had problems when I tried writing something in C. Specifically, I couldn't find
the nvml.h header file.

Turns out it lives in the [Tesla Deployment Kit](https://developer.nvidia.com/tesla-deployment-kit), along with a handy
program called nvidia-healthmon which fits neatly into your NHC or prologue/epilogue scripts for doing 
GPU sanity checks.  You'll also need nvml.h if you want to do something like build Torque with NVIDIA GPU support.
(Torque's website completely fails to mention you need the TDK, incidentally.)

OK, public service announcement achieved. Back to hacking... :)
