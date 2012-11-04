---
layout: post
title: "Some notes on the HPC cluster software stack"
date: 2012-11-03 18:48
comments: true
categories: hpc linux sysadmin  
---

__In this post, I'm mostly organizing a set of notes I've been using to help
people put together small high-performance computing clusters.__

For each category I'll try to summarize 
what that layer accomplishes for the cluster as a whole 
and give a few examples of software packages which fit into
that layer. I'm not going to try to come up with an exhaustive list for each
category, but instead talk about some packages which I've worked with 
personally and indicate a few favorites. I'll also mostly be
limiting myself to the Linux HPC world, deploying onto commodity hardware
(not a pre-built solution like a Cray), and mostly talking about open-source
solutions (because I'm often giving advice to poor graduate students).

The levels of the software stack I discuss include:

* [Provisioning System](#provisioning)
* [Configuration Management](#configuration)
  * with some notes on [ad-hoc tools](#adhoc) that are useful
* [Message passing libraries](#mpi)
  * with some notes on [library management](#modules) with environment-modules
* [Job scheduler](#scheduling)
* [Shared filesystems](#filesystem)
* [Monitoring](#monitoring)

<!-- more -->

### <a id="provisioning"></a>Provisioning system ###

An HPC cluster typically includes a few different types of computers:

* A master node, which runs any scheduler, monitoring, or other 
"infrastructure" software you have in place.
* One or more compute nodes which accomplish the actual computation.
* (optional) One or more separate fileserver nodes which export a shared
filesystem.
* (optional) One or more separate login nodes, which users log into to
submit their jobs.
* (optional) Other service nodes, which sometimes break out the functions of
a master node into multiple servers in a large cluster.

Now you could manually install an operating system on each of these
servers, put together each server's software stack, and carefully configure
each one... but for anything more than two or three compute nodes, this becomes
really unpleasant. So you typically want to have some method 
to automatically deploy a pre-configured system image to one or more identical
nodes.

If you're using a cloud service like Amazon's 
[Elastic Compute Cloud](http://aws.amazon.com/ec2/), the provisioning system is
provided with the service. For example, Amazon lets you boot up a VM from
an image provided by Amazon or the community, make changes, and then save your 
modified system to a new image you can re-deploy to mutliple new VMs.

If you're deploying onto hardware you own, there are a number of software packages
which can automate OS deployment for you. A few of the ones I've used to set up
HPC systems are:

* [Kickstart](http://en.wikipedia.org/wiki/Kickstart_(Linux)) is a method for
automating the installation of a Linux system using a file which specifies the
system configuration and a list of packages to be installed. You boot the system
with an installer for your Linux distribution of choice, and the installer 
follows the instructions in the kickstart file rather than
making you manually step through the install. Kickstart was designed for Red Hat
clones, but can be used for [Ubuntu](http://www.linuxuser.co.uk/tutorials/unattended-ubuntu-installations)
and other distros as well.

    The major advantage of using Kickstart is that it's just an installation file, 
and can be used whether your install DVD is a physical DVD, an ISO distributed
over the network, or whatever. A lot of provisioning systems require booting
over the network using [PXE](http://en.wikipedia.org/wiki/Preboot_Execution_Environment),
which works for most servers but is occasionally unavailable on older or 
desktop computers.

* [Cobbler](http://cobbler.github.com/) is a companion project to Kickstart, and
makes it easier to manage Kickstart images and boot them over a network. It also
includes a handy tool called koan which makes it possible to do network re-installs
even if you can't do automated PXE boots.

* [Warewulf](http://warewulf.lbl.gov/trac) is a cluster management system which is 
designed for HPC. It provides a number of tools for building and deploying
provisioning images, managing network configuration, and provides a basic configuration
management system (see below). 

    Like xCAT and some other HPC-focused tools, it 
allows both "stateful" provisioning (where Linux is installed to the local disk), and
"stateless" provisioning where the entire OS is simply loaded in RAM. This has the 
advantage of making it easy to change cluster configurations quickly because 
re-provisioning is much faster than writing the image to disk; but it means that
in stateless mode, the nodes can't reboot unless the master is available to
re-provision them.

    In the most recent releases, Warewulf also includes a package called 
"warewulf-cluster" which sets up some sane defaults for managing users, NFS, etc.
and "warewulf-icr", which is cerified under the 
[Intel Cluster Ready](http://software.intel.com/en-us/cluster-ready) program.

* [OpenStack](http://www.openstack.org/) is a tool for creating private 
"cloud"-style systems, in which virtual machines are provisioned onto a 
pool of hardware and managed in a self-service manner by the users. I have
qualms about using cloud-style provisioning systems for HPC because of 
the virtualization penalty and limited support for Infiniband, but 
OpenStack has a big community and provides some very good tools for 
image and node management.

* [Platform HPC](http://www.platform.com/) is a proprietary HPC cluster
management system which aims to build clusters in a "turn-key" fashion.
It does a pretty good job of this, provisioning stateful nodes using
a Kickstart-based system, and automates the configuration of networking.
It also integrates well with Platform's scheduler (LSF), provides some
monitoring, and functions well as an all-around management system.

Some other common provisioning systems designed for HPC 
include [xCAT](http://xcat.sourceforge.net/) and [Bright Cluster
Manager](http://www.brightcomputing.com/Bright-Cluster-Manager.php) (proprietary).

**My tool of choice:** Warewulf, because it's free, flexible, and it fits my
own mental model of clusters better than most other tools. I've also done 
a small amount of development for the project, so I'm very familiar with the code.


### <a id="configuration"></a>Configuration management ###

Provisioning "golden images" is a good way to deploy a large cluster of
identical systems, but it's not the most flexible system: if all you have is a 
set of images, you either need to re-provision whenever you want to make a change
(leading to downtime), or you need to make changes manually in-situ and carefully
sync everything back into the image.  

Configuration management systems
are used to bridge this gap, by providing an automated solution
for deploying configuration files and scripted installations after the nodes
are provisioned. Changing the configuration doesn't require a full OS 
re-deployment or a reboot, just re-running the configuration manager.

How much of your configuration you want to keep in a provisioning image vs
a configuration management system is a matter of choice. Some people
try to put everything in their OS image to simplify deployments, where others
provision pristine images straight from the Linux distribution and install their
entire HPC configuration using a configuration management system. 

My own approach
is somewhere between these extremes: I tend to provision system images which include 
all the software packages I want installed, but then deploy all my configuration files
using a configuration manager. This lets me re-use the provisioning image whenever I
want to set up a new cluster (because there's no cluster-specific configuration in 
the image), but means my configuration manager doesn't get bogged down
doing software installs.

Some configuration management systems I've used include:

* [Warewulf file provisioning](http://warewulf.lbl.gov/trac/wiki/Provision/Files):
On Warewulf-provisioned clusters, I use the "file provisioning" system to sync
configuration files to compute nodes. It's an extremely lightweight system: the
files and their metadata are stored in a database on the master node, includes
a basic templating language, and the files are served over HTTP. The nodes don't 
have any kind of specialized daemon on them to keep in sync: instead, there's just
a basic script that's run by a cron job to download updated files using wget every
five minutes. 

    The limitation of this system is that it can't do everything a larger
configuration manager can do. File provisioning isn't really useful for 
provisioning software packages or managing service daemons, it just does
what it says on the tin: provisions files. I find it usually works extremely
well with systems like compute nodes, but it can be limiting when you need a 
more flexible system.

* [Puppet](http://puppetlabs.com/) is a configuration management system which
defines a domain-specific languange (DSL) for describing the configuration of
a server. For example, to install and configure the Torque scheduler client on
a compute node, a snippet might look like this:

        class torque_mom {
            package { "torque-client":
                ensure => installed
            }

            file { "mom_config":
                path => "/var/spool/torque/mom_priv/config",
                ensure => file,
                require => Package['torque-client'],
                source => 'puppet://modules/torque_mom/config'
            }

            service { "pbs_mom":
                name => "pbs_mom",
                ensure => running,
                enable => true,
                subscribe => File["mom_config"]
            }
        }

    This manifest would then be read by the puppetd daemon on each node,
    which would apply the described configuration to the server.
    Puppet manifests can be run in any order, so you have to specify dependencies 
    manually where they exist. This helps ensure that the manifest is 
    [idempotent](http://en.wikipedia.org/wiki/Idempotence), meaning that you can
    run the same manifest multiple times and the final state will always be the 
    same. This is helpful for maintaining a consistent system, because if a 
    node gets in a weird state you can just re-run the puppet manifests to 
    restore it to a good state.

* [Opscode Chef](http://www.opscode.com/chef/) uses a DSL as well, but this DSL
is really some specialized syntax added to the Ruby language. Chef recipes are 
distributed to client nodes and run as Ruby scripts, but let you define 
configuration files, software resources, and services like this:

        package "torque-client" do
            action :install
        end

        cookbook_file "mom_config" do
            source "mom_config"
            action :create
            path "/var/spool/torque/mom_priv/config"
        end

        service "pbs_mom" do
            supports :status => true, :restart => true
            action [:enable, :start]
        end

    It looks a lot like Puppet, but you can use the rest of the Ruby language too:
    loops, conditionals, etc. It also doesn't do any re-ordering or dependency 
    management: all the recipes run from top to bottom, in order of their filenames.
    This makes it easier to think through how a given recipe execute, but makes
    ensuring idempotence trickier.

    I like using Chef for building systems like master nodes, login nodes,and 
    other systems which are harder to build using Warewulf  Its 
    "Ruby with extensions" style makes it more flexible than Puppet (in my opinion),
    but also less predictable. But I don't like how much supporting infrastructure
    it takes to run on a node (specific versions of Ruby and many supporting gems), 
    so I don't typically use it on compute nodes.

Other configuration management systems, which I don't have personal experience
with, include [Ansible](http://ansible.cc/) and [Cfengine](http://cfengine.com/).

**My tool of choice**: A combination of Warewulf for compute nodes and Chef
for repeatable infrastructure.

#### <a id="adhoc"></a>Ad-hoc management tools ####

Another category worth noting here is "ad-hoc" cluster 
management tools: systems which
are more about carrying out operations on a large number of nodes 
simultaneously, than about ensuring a consistent system state. Two useful tools
I've encountered are:

* [pdsh](http://code.google.com/p/pdsh/): the Parallel Distributed Shell, all it 
does is execute ssh commands in parallel on a large number of nodes. This is 
really useful for doing quick on-the-fly operations on a cluster. For example,
running `pdsh -w node[000-999] "touch /tmp/file"` will create a file called 
"/tmp/file" on all thousand nodes in a cluster. It's just that simple.

* [Fabric](http://docs.fabfile.org/en/1.4.3/) is a Python library which accomplishes
similar operations as pdsh, basically doing ssh in a loop. The difference is that
you typically write a python script to accomplish a series of operations, like so:

        from fabric.api import local

        def prepare_deploy():
            local("./manage.py test my_app")
            local("git add -p && git commit")
            local("git push")

    (borrowed from the Fabric tutorial). Running `fab prepare_deploy` would then run the relevant
    commands on all the nodes supplied, either on the command line or in a config file.

### <a id="mpi"></a> Message passing libraries ###

In cluster computing, the class of libraries that gets the most attention is
generally the [Message Passing Interface](http://en.wikipedia.org/wiki/Message_Passing_Interface)
(MPI). MPI is a standardized interface which simplifies inter-process communication 
for parallel applications. Most popular applications for clustered scientifc computing 
use MPI. 

In case you haven't seen much MPI code, here's
a simple example of an MPI prorgram which includes communication between
processes, where each process sends a greeting message to the rank-0 process:

    #include <stdio.h>
    #include <mpi.h>
    #include <string.h>

    int main(int argc, char* argv[]) {
        int rank, count, source, dest;
        int tag = 0;
        char message[100];
        MPI_Status status;

        MPI_Init(&argc, &argv);
        MPI_Comm_rank(MPI_COMM_WORLD, &rank);
        MPI_Comm_size(MPI_COMM_WORLD, &count);

        if (rank != 0) {
            sprintf(message, "Greetings from process %d!",my_rank);
            dest = 0;
            MPI_Send(message, strlen(message)+1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
        } else {
            for (source = 1; source < count; source++) {
                MPI_Recv(message,100,MPI_CHAR,source,tag,MPI_COMM_WORLD, &status);
                printf("%s\n",message);
            }
        }
        MPI_Finalize();
    }

This program can be run across multiple systems, on whatever interconnect you like,
without thinking about any of the networking involved: MPI abstracts the communication
details away from you, leaving you to focus on the (already difficult) problem of
parallel computing. Just run `mpirun -np <number-of-processes> -hostfile <list-of-hosts> ./a.out`.
(Syntax can vary by implementation.) It provides a lot of useful constructs not just for sending and
receiving messages, but providing barriers for synchronization, topologies
for thinking about your processes in terms of your problem decomposition, etc.

*It's worth noting that MPI programs depend on the assumption of a reliable network,
and typically don't have any resiliancy against major network or node failures.
This is in contrast to "Big Data" systems like Hadoop.*

However, "MPI" isn't a software package: it's a standard defined by committee, and
there are multiple competing implementations of this standard, both
open-source and proprietary. These implementations
often specialize for certain hardware, or for different types of performance. A 
few MPI implementations worth knowing about include:

* [OpenMPI](http://www.open-mpi.org/): One of the most popular MPIs out there, it
is open-source, integrates with many different job schedulers, and supports most 
different cluster interconnects with good performance. 
* [MPICH2](http://www.mcs.anl.gov/research/projects/mpich2/): Developed by
Argonne National Lab, MPICH2 is almost as widely used as OpenMPI. Its usage 
model is a little different than OpenMPI, but is also well-supported by 
most schedulers.
* [MVAPICH2](http://mvapich.cse.ohio-state.edu/overview/mvapich2/) is an MPI
based on MPICH2, specialized for use with the high-performance 
Infiniband interconnect. Recently it has also included integration with CUDA
for doing direct memory copies of data in GPU memory for NVIDIA GPUs.
* [Intel MPI](http://software.intel.com/en-us/intel-mpi-library) integrates well with the Intel
compilers and offers generally high performance. The newest versions also have
support for Intel's new Xeon Phi accelerators.

There are many other implementations, including a lot of specialized and proprietary 
implementations: the ones above are just the ones I'm most familiar with.

Choosing an MPI is complicated by whether you are writing your own software
or using an open-source or licensed application; what hardware you will be using,
especially what interconnect you are using; which HPC scheduler you're using, if any;
and many other factors. My advice is to use the MPI recommended by your software vendor,
which works with your hardware, or just using your favorite.

### <a id="modules"></a>Library management: Environment Modules ###

On many clusters, including almost all shared systems, you can't choose just one 
implementation of common libraries such as MPI, BLAS, LAPACK, etc. You might
also want to have multiple versions of the same compilers or tools available,
for application compatibility and performance tests. For example, I work with a couple
of applications which only work with older versions of OpenMPI, but I still want to use
the new versions on the same cluster. 

However, it's often
hard to keep multiple versions of the same software around on Linux, as they tend
to want to own the same files. One solution to this problem is the 
[environment modules](http://modules.sourceforge.net/)
system. 

When using environment modules, you typically install each software package into
a non-standard location. For example, instead of letting each of your MPI implementations
install into the standard Linux locations (`/usr/bin`, `/usr/lib`, etc), you install each 
one into a self-contained folder. For example, we might install OpenMPI 1.6.2 into
`/opt/openmpi-1.6.2`.

We then define a *modulefile* for each software package. The modulefile defines changes
to the user's environment variables which are required to use the software package
in question. For our OpenMPI package, the modulefile might look something like 
this:

    #%Module
    set     root            /opt/openmpi-1.6.2
    prepend-path    PATH                    $root/bin
    prepend-path    LD_LIBRARY_PATH         $root/lib
    prepend-path    C_INCLUDE_PATH          $root/include
    prepend-path    MANPATH                 $root/share/man
    conflict mpi

Let's go line-by-line. The first line declare this to be a module-file; the next
line defines a "root" variable which shows where the software is installed. The next
four lines use the "prepend-path" command to add the OpenMPI bin, library, include, and 
man directories to the relevant environment variables as the first entry; and the "conflict
mpi" line notes that this modulefile conflicts with other mpi modulefiles.
We then put this file in the modulefiles directory (/usr/share/Modules/modulefiles) as
`mpi/openmpi/1.6.2`. 

On my personal development system, I have this and other modules installed to manage my 
software. If I type "module avail", I see the following output:

    [ajdecon@exp ~]$ module avail
    
    ----------------------------------------------------------- /usr/share/Modules/modulefiles -----------------------------------------------------------
    dot                     module-info             mpi/openmpi/1.6.2       python/2.7.3            use.own
    gcc/4.7.2               modules                 mpi/openmpi/1.7-current python/3.2.3
    module-cvs              mpi/mpich2/1.4.1        null                    ruby/1.9.3-p194

So you can see that I have multiple conflicting MPI and Python versions installed, as well as some 
other software. Then, when I load my OpenMPI 1.6.2 module, it changes my PATH to make sure I 
point to the right files:

    [ajdecon@exp ~]$ echo $PATH
    /usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/ajdecon/bin
    [ajdecon@exp ~]$ module load mpi/openmpi/1.6.2 
    [ajdecon@exp ~]$ echo $PATH
    /opt/openmpi-1.6.2/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/ajdecon/bin

### <a id="scheduling"></a>Cluster scheduler ###

HPC clusters are often expensive systems, and it's important to make efficient use 
of them. This is especially true on a shared system where multiple users are
competing for the same resources. HPC cluster schedulers generally implement
a queuing system, where the compute nodes are divided into one or more queues
and jobs are submitted to the scheduler. These jobs are then run automatically 
by the scheduler when the required resource become available.

In most schedulers, this sort of automation is accomplished using the concept
of a job script. Each user writes a simple (or not-so-simple) program, usually
a shell script, automates the process of running their job. This script often
contains directives to the scheduler which describe what resources are 
required. It is this program which is run by the scheduler when the job reaches
the head of the queue.

For example, a script for the PBS scheduler to run an MPI program which requires
a temporary data directory might look like this:

    #!/bin/bash
    #PBS -l nodes=4:ppn=12
    #PBS -l walltime=02:00:00
    mkdir /tmp/data
    cd $HOME/myprogram
    mpirun -np 48 ./myprogram --datadir=/tmp/data

This script includes directives to the scheduler saying that it needs 4 nodes with
12 processors per node and that it will need 2 hours to run. It creates its data directory,
changes to the directory where the binary is located, and launches a 48-process MPI 
program.  In most schedulers, there is some mechanism for the STDOUT and STDERR 
of the job to be captured and saved for the user, usually as files in the user's
home directory identified by job number.

HPC schedulers also generally provide some facilities for managing the compute
nodes they will use to run: reporting on the CPU and memory activity, identifying
which nodes have GPUs installed, and other resource management features. They
also usually have the concept of "offlining" or "draining" a node, marking 
an individual compute node so that it will be assigned no new jobs. This allows 
the system administrator to let a node finish any existing jobs, then do 
maintenance on the node without worrying about disturbing new users.

A few common HPC schedulers you might use on a cluster are:

* [Torque](http://www.adaptivecomputing.com/products/open-source/torque/): Based on 
the old OpenPBS scheduler, Torque is a common open-source HPC resource manager 
developed by Adaptive Computing. It provides various facilities for node 
management and a simple "first-in first-out" scheduler. Torque also has 
extremely good integration with many MPI implementations, so that an MPI 
program can get its host-list directly from the scheduler with no
incantations by the user.

    * Adaptive Computing also develops the open-source [Maui](http://www.clusterresources.com/products/maui-cluster-scheduler.php)
      and commercial [Moab](http://www.adaptivecomputing.com/products/hpc-products/moab-hpc-basic-edition/)
      schedulers. These schedulers "sit on top" of a resource manager like Torque,
      providing more advanced options for scheduling user jobs. These products
      can be used to implement quality-of-service options for specific users;
      implement "fair-share" scheduling in which users who have not had any
      recent allocations get higher priority; and many other options.
      I tend to put together a lot of clusters which run Torque with Maui
      as the scheduler, but Moab has even more advanced features (and is
      updated more often).

* [Grid Engine](http://en.wikipedia.org/wiki/Oracle_Grid_Engine) is a 
popular scheduler with a complicated past. Originally developed by Sun 
Microsystems, it is went with the rest of Sun's IP to Oracle... except
that it was also an open-source project, which was forked to the name 
[Open Grid Scheduler](http://gridscheduler.sourceforge.net/) when the 
community became dissatisfied with Oracle's stewardship. Meanwhile 
[Univa](http://www.univa.com/products/grid-engine) hired many of Sun's
original Grid Engine developers and established their own commercial
fork, and this inspired [Son of Grid Engine](https://arc.liv.ac.uk/trac/SGE),
yet another open source fork.

    Confused yet?

    For all its complicated history, Grid Engine is a high-quality and popular
    HPC scheduling system. It's a little trickier to configure than Torque
    (in my opinion) and manages its queues differently, but it fundamentally
    manages the same problems. It also provides better quality-of-service
    and prioritization options than the built-in Torque scheduler, though I 
    don't think it quite matches Maui or Moab.

    Another noteworthy detail about Grid Engine is that it's the scheduler 
    of choice for the [MIT StarCluster](http://star.mit.edu/cluster/) project,
    which provides easy automation for setting up an HPC cluster on Amazon's 
    EC2 service. If you want to run on EC2, you could do worse than to just
    run StarCluster.

* [SLURM](http://www.schedmd.com/slurmdocs/slurm.html) is another open-source 
resource manager, originally developed by Lawrence Livermore National Lab. It's
an extremely scalable solution, able to run on truly huge supercomputing clusters,
and has a lot of useful new ideas on resource management. It's also a lot easier to
configure than Torque or GridEngine (in my opinion), but has less in the way of 
easy MPI integration, mostly because it's newer. SLURM's built-in 
scheduler is also FIFO like Torque, but can also integrate with Maui or Moab for
more complex quality-of-service rules.

Some other schedulers which are in common use include 
[Platform LSF](http://www.platform.com/workload-management/high-performance-computing) and
[PBS Professional](http://www.pbsworks.com/Product.aspx?id=1&AspxAutoDetectCookieSupport=1) by
PBS Works.

**My tool of choice**: As a sysadmin I typically prefer using SLURM because of its ease of cofiguration,
but PBS-based systems like Torque are much more common and most users are more
familiar with them at this time.

### <a id="filesystem"></a>Shared filesystem ###

Most HPC clusters make use of shared network filesystems. These are typically used for user
home directories, shared software, and sometimes for fast shared "scratch" filesystems for
temporary job files. A shared filesystem is often the most brittle part of an HPC 
cluster, as these systems tend to fail more often than schedulers or MPI communication,
but are so useful it's probably still worth it.

Most small HPC clusters should just use [NFS](http://en.wikipedia.org/wiki/Network_File_System): 
it provides decent performance and
reliability, is built in to most Linux distributions, and is very easy to set up.
My advice in most cases is to set up your cluster with NFS first and benchmark
applications. If you can get away with it, stop here: it all becomes much more complicated
from there.

However, the name of the game is "high performance", and many applications become I/O 
bound if run with a slow NFS server; so there are several parallel
filesystems used on HPC clusters to eliminate I/O as a performance blocker.

The only one I'm really familiar with is [Lustre](http://wiki.lustre.org/index.php/Main_Page),
a shared filesystem which
achieves high-performance by striping across disks attached to multiple I/O nodes.
With a fast network, this improves performance both by increasing the number of disks
any file is striped across, and by sharing the load across multiple network connections
on multiple nodes. Lustre achieves this high performance in part by working at the level
of the Linux kernel, and requires a patched kernel for the I/O nodes. 

One interesting feature of Lustre is that it actually allows the user to
set up how any given file or directory is striped across the I/O nodes,
so the particular I/O patterns can be tuned for any given job or application.

Lustre, like Grid Engine, is an old Sun Microsystems project that has since
been somewhat neglected by Oracle. Much of the interesting work on Lustre
has recently been done by a startup called [WhamCloud](http://www.whamcloud.com/),
who also sell some useful management tools for Lustre filesystems.

Other parallel filesystems include [PVFS2](http://www.pvfs.org/), IBM's
[GPFS](http://www-03.ibm.com/systems/software/gpfs/), and 
[GlusterFS](http://www.gluster.org/).

**My tool of choice:** NFS if I can get away with it, otherwise Lustre.

### <a id="monitoring"></a>Monitoring system ###

My tools of choice for monitoring HPC clusters include:

* [Ganglia](http://ganglia.sourceforge.net/) for real-time monitoring of
cluster usage. Ganglia monitors just about everything: CPU, memory, networking, GPUs,
and many other metrics. If a process is running away with too many resources, you
can probably see it in Ganglia.

* [Nagios](http://www.nagios.org/) for notifications of problems like down nodes,
full filesytems, dangerous loads, etc. Nagios can be tricky to learn to configure,
but is extremely extensible and gan monitor just about anything with a little work.

### My preferred cluster stack ###

Just to summarize at the end: here is my own preferred stack, subject to change
based on the needs of the particular situation.

* __Warewulf__ for provisioning
* __Warewulf__ and __Chef__ for configuration management
* __OpenMPI__ for MPI, or whatever your app works best with
* __Environment modules__ for managing different libraries and compilers
* __SLURM__ for job scheduling
* __NFS__ for a shared filesystem, __Lustre__ if I need the performance
* __Ganglia__ and __Nagios__ for monitoring.

But the right answer is to always benchmark, profile, and talk to your users!
