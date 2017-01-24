---
layout: default
title: GPU Hardware
permalink: /hardware/
---

# Hardware Overview #

NVIDIA GPUs are manufactured under a range of product families all of which support CUDA. The cheapest desktop GPUs are the GeForce range which are consumer cards primarily designed for gaming workstations. GeForce GPUs are excellent value for money by lack some advanced functionality (ECC memroy correction and high double performance) of the more expensive product ranges. GeForce GPus can be run in server environments but this is generally not encouraged by NVIDIA. The Quadro range of cards is more expensive than the GeForce range and are designed for professional graphics workstations, as such they have features and support which are advantageous for professional graphics rendering. As their CUDA capabilities represent poor value for money in comparison to other products they are rarely recommended for GPU computing (unless you also require professional graphics). The Tesla range of GPUs is designed for professional GPU computing. Tesla GPUs are based on the same architectures are the GeForce and Quadro devices but are designed run compute rather than have displays connected to them. The Tesla GPUs have improved double precision support, are made form higher quality components and are warranted to be run in server solutions. Due to the advent of Deep Learning the Tesla range is now more specialised into general compute GPUs (e.g. K20, K40 and P100) and inference GPUs (e.g. M40, M80) which are designed to be run on production servers where deep learning networks are deployed.

For more in depth details on NVIDIA GPU product ranges take a look at one of the [CUDA training courses](./education) which cover architecturual details.

GPUs are also available from ATI at very competitive price points and with similar product ranges. These GPUs do not howwver support CUDA so care should be taken to ensure the software you intend to deploy on them has OpenCL support (see (software)[./software] section of the site).

Xeon Phis are an alternative to GPUs which also employ many core parallelism and vectorisation. These were previously designed as acclerator cards (with simmilar form factor to GPUs) but are now availble as sockets CPUs. Xeon Phis use 

## Sheffield GPU Facilities ##

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

## National Facilities ##

A number of EPSRC Tier 2 centre with GPUs will be available in Mid 2017. Details of which will be listed here.
