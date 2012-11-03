---
layout: post
title: "The HPC cluster software stack"
date: 2012-11-03 18:48
comments: true
categories: hpc linux sysadmin 
---

My work brings me into contact with a lot of people who are just getting
started in high-performance cluster computing. Often those first steps include
putting together a small HPC cluster for development or small-scale testing,
either on a few cheap servers or a cloud computing service; and since I'm 
a cluster sysadmin by trade, I get a lot of questions about how to put 
everything together.

Like every complex computing system, though, the software which runs an
HPC cluster tends to get put together as a stack of inter-related software
packages which each manage different parts of the system. Not every layer is 
necessary, depending on what type of cluster you're putting together,
but it's good to know what each one does.

In this post, I'm mostly organizing a set of notes I've been using to help
people put together small clusters. For each category I'll try to summarize 
what that layer accomplishes for the cluster as a whole, how it integrates with
the other layers, and give a few examples of software packages which fit into
that layer. I'm not going to try to come up with an exhaustive list for each
category, but instead talk about some packages which I've worked with 
personally and maybe indicate a few favorites.

<!-- more -->

### Provisioning system ###



### Configuration management ###

#### Ad-hoc management tools ####

### Cluster scheduler ###

#### Resource manager vs scheduler ####

### Libraries for parallelism ###

#### Library management: Environment Modules ####

### Monitoring system ###

### Proprietary all-in-one solutions ###

### My stack of preference ###
