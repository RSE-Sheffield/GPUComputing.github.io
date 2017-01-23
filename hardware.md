---
layout: default
title: GPU Hardware
permalink: /hardware/
---

# Hardware Overview #


# Cloud Based Resources #

Azure and Amazon Ec2. Grants etc.

# Local Hardware Options #

## Local GPU Facilities ##

ICEBERG: The University of Sheffield has a local HPC Facility called Iceberg. This has a limited number of  GPUs. 

ShARC: The successor to Iceberg called ShARC (the Sheffield Advanced Research Computer) is due to be on-line in October 2016. It will contain significantly more GPUs (16 x dual PCB NVIDIA K80s) and has a shared computing model (see section on Purchasing GPUs) which has an additional 8x dual PCB NVIDIA K80s and a state of the art  NVIDIA DGX-1 system with 8x NVIDIA P100s. Shared computing contributions are available to everyone however purchases have priority access. See [full system specifications](./todo)

Documentation on accessing both of these facilities is maintained on the [Sheffield HPC documentation](http://rcg.group.shef.ac.uk/iceberg/index.html) website.

### Purchasing GPU Workstations ###

The university has a framework agreement for workstation PCs with Dell and Viglen. Some of the machines which can be purchased from these providers have reasonable GPUs. For a more configurable and upgradeable option it is possible to purchase a workstation machine under the specialist equipment rules. The university has a number of hardware components providers (if you want to build your own machine) or a number of companies which provide specialist GPU workstation machines. Scan computing has a university account manager ([Kerry Ozard](mailto:kerryo@scan.co.uk)) who can help with purchasing GPU machines. 

It is possible to obtain free GPUs from NVIDIA via the [hardware donations program](https://registration.nvidia.com/ahr.aspx). If you are interested in this you must provide your own system (workstation) to host the provided hardware. This must have a sufficiently large power supply, usually 2x 8 pin PCI-E connections and plenty of space. If you are thinking of applying for this program then its is advisable to speak to Paul Richmond to get help writing an application.

### Purchasing Server GPUs ###

NVIDIA Tesla GPUs are available with a 15% discount via the Sheffield NVIDIA GPU Research centre program. 

### The GPU Shared Facility ###

For larger GPU purchases CICS are committed to working with RSE Sheffield to build a shared computing model. The exact details of this are still to be finalised but the shared model has the following features;

* It will be possible to purchase GPU server racks (GPU nodes) in the form of either Dell C4130s with up to 4 Tesla GPUs (usually K80s) or DGX-1s with 8x P100 GPUs
* CICS will integrate GPU nodes into ShARC and support the nodes for the lifetime of the ShARC system
* Software installations and support will be managed by CICS and RSE Sheffield.
* You will have priority access to your own nodes but when not in use these will be available within the general ShARC pool (this is different to exclusive access).

If you are interested in purchasing GPU servers then please contact [RSE Sheffield](http://www.rse.shef.ac.uk/contact) who can advice on purchasing options and arrange this via CICS.
