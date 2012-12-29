---
layout: page
title: "Projects"
comments: false
sharing: false
footer: true
---

## HPC ##

* [torque_qhistory](http://blog.ajdecon.org/torque_qhistory/): A 
tool for viewing and doing simple queries on PBS/Torque accounting files.

* [modules_logger](http://blog.ajdecon.org/modules_logger/): A system for logging 
the usage of environment-modules to help system administrators better manage 
cluster software. Uses MongoDB as the persistence layer.

* [hpc-helper-scripts](http://blog.ajdecon.org/hpc-helper-scripts/): 
This is just an assortment of scripts, notes, and miscellaneous tools I've put together for helping with managing HPC clusters. Some of these are trivial, others are decent tools in their own right;
they're included here in case they are useful to others.

* [Warewulf](http://warewulf.lbl.gov): a system for provisioning and managing 
Linux systems, well-optimized for working with HPC clusters, and developed
out of Berkeley National Lab. I'm a minor contributer to the project, and have
done a little
work on the stateful provisioning, IPMI and user interface, and mostly done a
lot of bugfixing as I run into problems. One of my primary tools as an HPC admin.

* [hpc-chef](https://github.com/ajdecon/hpc-chef): A Chef repo with some 
cookbooks related to HPC (and some other experiments). Woefully out-of-date
at the moment, but occasionally new things filter in.

## From grad school ##

* [imagej_morphology](https://github.com/ajdecon/imagej_morphology): A library
based on the popular [ImageJ](http://rsb.info.nih.gov/ij/) image processing 
program for carrying out morphological operations, and some associated plugins
for ImageJ itself.

* ["Genotyping by Alkaline Dehybridization Using Graphically Encoded 
Particles"](http://colloids.matse.illinois.edu/articles/zhang_ChemEurJ.pdf): This 
project, led by Huaibin Zhang, demonstrated a cool way to do DNA discrimination
using pH as a driving force rather than heat, and encapsulated it in a neat
polymer-particle and microfluidic device. I was involved in the fabrication
of the encoded microparticles, and in the image processing of the microscopy
images and some of the subsequent analysis. The 
[supplementary information](http://www.ajdecon.org/pubfiles/Zhang-et-al_Supplementary_Information.pdf)
on this paper is worth reading if you enjoy the analysis portion.

* [M.S. thesis](https://github.com/ajdecon/ms_thesis): "Fabrication, dynamics and 
self-assembly of anisotropic colloidal particles." Details my work on the fabrication
of "patchy" microparticles with non-spherical geometry and non-uniform composition, 
and some work on measuring and analysing their interaction at low concentration.
Really only of interest if you enjoy soft matter materials science, but analyzing
the microscopy data was also
a good "practical" application of morphological image processing.

 - [grad school matlab](https://github.com/ajdecon/gradschool_matlab): A bunch
   of the matlab scripts I used for processing data for my thesis. Like much
   academic code it's uniformly horrible, but it might be worth reading and
   re-writing if you're doing related work. Licensed under the
   [CRAPL](http://matt.might.net/articles/crapl/)

Also, see a list of my [academic publications](/pubs).
