---
layout: post
title: "Building tiny compute clusters for fun and learning"
date: 2013-07-20 09:54
comments: true
categories: HPC raspberry_pi
---

<img src="https://s3.amazonaws.com/ajdecon-public/Pi_Lego_detail.jpg" width=600>

Just for fun, 
I recently built a small HPC-style computing cluster out of five 
[Raspberry Pi](http://www.raspberrypi.org/) servers.  While this kind of 
thing has of course been [done](http://www.southampton.ac.uk/mediacentre/features/raspberry_pi_supercomputer.shtml)
[before](http://coen.boisestate.edu/ece/raspberry-pi/), I wanted to build one 
anyway as a geek toy and a platform for me to play with a few HPC ideas.

I think it actually turned out rather well, and it was a ridiculous amount of
fun to build. My wife [Leigh](http://www.twitter.com/Leigh_47) helped put together
the Lego rack, and it was a lot of fun to put together. Because I was focusing more on
duplicating a "real" cluster rather than getting performance, I included all the usual
cluster services such as a [SLURM](http://slurm.schedmd.com) scheduler, 
[Ganglia](http://ganglia.sourceforge.net) for monitoring, and using one of the
Pi's as a non-computing login node. Total [HPL performance](http://gist.github.com/ajdecon/6045818),
670 MFlops. Oh yeah!

A few folks at [NVIDIA](http://www.nvidia.com/), where I also work on HPC clusters, 
also thought this was a cool project. The Pi cluster thus ended up being featured
as an "Inner Geek" post on the 
[company blog](http://blogs.nvidia.com/blog/2013/07/17/raspberry-pi/), alongside 
other awesome projects from NVIDIANs such as a
[cool airplane restoration](http://blogs.nvidia.com/blog/2013/05/23/inner-geek-restoring-my-fathers-meyers-aero-commander/)
and an [amazing holiday lights display](http://blogs.nvidia.com/blog/2012/11/28/inner-geek-nvidia-engineer-serves-up-holiday-lights-gangnam-style/).

If you're curious, I have details on how I put it together in my 
[ansible-pi-cluster](https://github.com/ajdecon/ansible-pi-cluster) GitHub
repository. This includes the [Ansible](http://www.ansible.cc) playbooks
I used to install all the software (though they still need some work), as
well as some notes on the physical assembly and OS provisioning in the
[README](https://github.com/ajdecon/ansible-pi-cluster/blob/master/README.md).

I'll admit that the [performance isn't awesome](http://gist.github.com/ajdecon/6045818) 
compared to other clusters 
(or even many laptops!), but I like having a cluster I can hold in my hands:

<img src="https://s3.amazonaws.com/ajdecon-public/PiWithLegos.jpg" width=600>


There are all sorts of other "tiny HPC cluster" projects out there,
frequently used for teaching, and which have often done some real science.  
Here's a short roundup of some other tiny clusters I've seen...
And if you know something else I should include in this post, let me know
and I'll add it in.

* [Idris-Pi](http://www.southampton.ac.uk/~sjc/raspberrypi/),
the original Raspberry Pi + Legos supercomputer from Southampton University.
This cluster included 64 Raspberry Pi boards, and they also have some 
[instructions](http://www.southampton.ac.uk/~sjc/raspberrypi/pi_supercomputer_southampton.htm)
posted for how they set up the software stack.  There was also a very
interesting discussion of this cluster on the 
[Beowulf list](http://www.beowulf.org/pipermail/beowulf/2012-September/029953.html)

<iframe width="560" height="315" src="//www.youtube.com/embed/Jq5nrHz9I94" frameborder="0" allowfullscreen></iframe>

<br/><br/>
* [RPiCluster](http://coen.boisestate.edu/ece/files/2013/05/Creating.a.Raspberry.Pi-Based.Beowulf.Cluster_v2.pdf)
from Boise State, a 32-node cluster built by [Joshua Kiepert](http://www.linkedin.com/in/jkiepert)
to use for his dissertation research. This link also includes detailed instructions on building
his cluster.

<iframe width="560" height="315" src="//www.youtube.com/embed/i_r3z1jYHAc" frameborder="0" allowfullscreen></iframe>

<br/><br/>
* [LittleFe](http://littlefe.net/) is an ongoing project focused on building
small educational HPC clusters which teachers can use in their classes.
Their web site includes a continually-updated design for a 6-node cluster
of Intel Atom processors, and their organization also holds build events at 
conferences such as [SuperComputing](http://www.supercomp.org/) where
interested teachers can build clusters they can take home.

<iframe width="560" height="315" src="//www.youtube.com/embed/ciY4vsij_ak" frameborder="0" allowfullscreen></iframe>
<br/><br/>

-----------------------------

