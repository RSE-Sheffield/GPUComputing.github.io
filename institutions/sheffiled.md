---
layout: page
title: GPUComputing@Sheffield (The University of Sheffield)
permalink: /Sheffield/
---

![The University of Sheffield](/static/img/TUOS.png){: .floatright}

## About ##

The GPUComputing@Sheffield group is a virtual group with a focus on GPU and accelerated computing. The group has been an NVIDIA GPU research centre since 2011 and an NVIDIA Education Centre since 2016 ([more history](http://paulrichmond.shef.ac.uk/gpu/gpucomputingatsheffield/)). The group is maintained mainly through a mailing list and webpage which has recently migrated to this [www.accleratedcomputing.ac.uk/Sheffield](http://www.accleratedcomputing.ac.uk/Sheffield) site to promote community contributions and to improve links with the national community.

## Research Software Engineering Support ## 

The recently formed Research [Software Engineering Sheffield (RSE Sheffield) group](http://www.rse.shef.ac.uk) has a number of staff which are able to provide research software consultancy for GPU computing within the University of Sheffield. These GPU specialists work under the guidance of [Dr Paul Richmond](http://www.prichmond.staff.shef.ac.uk) and undertake GPU software consultancy for industry as well as "limited" free research consultancy and advice.

## On-line Resources ##

Mailing List: GPUComputing@Sheffield has a [google group mailing list](https://groups.google.com/a/sheffield.ac.uk/forum/#!forum/gpucomputing) for local (Sheffield) GPU computing assistance and announcements. This mailing list can be used for

* Help and issues with GPUs on the local HPC facilities (Iceberg and ShARC)
* Questions and discussion relating to GPU enabled software (general software queries should be direct to ???)
* Announcements for training and education.

Twitter: Dr Paul Richmond ([@gpu_mondus](https://twitter.com/gpu_mondus)) makes announcements relating to GPU and accelerated computing training and software. Dr Mike Croucher ([@walkingrandomly](https://twitter.com/walkingrandomly)) also tweets more generally on Research Software Engineering including GPU computing at Sheffield.

## Training ##

Short Courses: GPUComputing@Sheffield organises a number of short one day courses on GPU computing (usually in March and November) which are advertised through the mailing list. These are prioritised for Sheffield researchers but will be advertised on the www.acceleratedcomputing.ac.uk [training](./training) calendar. The training material and lecture notes are [available on-line to anyone](http://paulrichmond.shef.ac.uk/teaching/CUDA/).

Parallel Computing with GPUs taught module: Dr Paul Richmond provides a Spring Semester undergraduate (COM4521) and post-graduate (COM6521) teaching module, [Parallel Computing with GPUs](http://paulrichmond.shef.ac.uk/teaching/COM4521/). This module is available to all undergraduate and postgraduate students with the necessary background requirements as well as University of Sheffield staff and research students. The course material is available on-line for anyone to browse (with the exception of the assessment material)

## Local GPU Facilities ##

ICEBERG: The University of Sheffield has a local HPC Facility called Iceberg. This has a limited number of  GPUs. Documentation on accessing this facilities and the GPU accelerated software supported is maintained on the [Iceberg documentation](http://rcg.group.shef.ac.uk/iceberg/index.html) website.

ShARC: The successor to Iceberg called ShARC (the Sheffield Advanced Research Computer) is due to be on-line in October 2016. It will contain significantly more GPUs (16 x dual PCB NVIDIA K80s) and has a shared computing model (see section on Purchasing GPUs) which has an additional 8x dual PCB NVIDIA K80s and a state of the art  NVIDIA DGX-1 system with 8x NVIDIA P100s. Shared computing contributions are available to everyone however purchases have priority access. 

### Purchasing GPU Workstations ###

The university has a framework agreement for workstation PCs with Dell and Viglen. Some of the machines which can be purchased from these providers have reasonable GPUs. For a more configurable and upgradeable option it is possible to purchase a workstation machine under the specialist equipment rules. The university has a number of hardware components providers (if you want to build your own machine) or a number of companies which provide workstation machines. Scan computing has a university account manager ([Kerry Ozard](mailto:kerryo@scan.co.uk)) who can help with purchasing GPU machines. 

It is possible to obtain free GPUs from NVIDIA via the [hardware donations program](https://registration.nvidia.com/ahr.aspx). If you are interested int his you must provide your own system (workstation) to host the provided hardware. This must have a sufficiently large power supply, usually 2x 8 pin PCI-E connections and plenty of space. If you are thinking of applying for this program then speak to Paul Richmond or the mailing-list to get help writing an application.

### Purchasing Server GPUs ###

NVIDIA Tesla GPUs are available with a 15% discount via the Sheffield NVIDIA GPU Research centre program. The best discount is actually available if you have taken part in an NVIDIA organised training session where there are 50% discount codes for participants. Ask on the [mailinglist](https://groups.google.com/a/sheffield.ac.uk/forum/#!forum/gpucomputing) as a number of people may have unused discount codes that they may be able to offer you.

### The GPU Shared Facility ###

For larger GPU purchases CICS are committed to working with RSE Sheffield to build a shared computing model. The exact details of this are still to be finalised but the shared model has the following features;

* It will be possible to purchase GPU server racks (GPU nodes) in the form of either Dell C4130s with up to 4 Tesla GPUs (usually K80s) or DGX-1s with 8x P100 GPUs
* CICS will integrate GPU nodes into ShARC and support the nodes for the lifetime of the ShARC system
* Software installations and support will be managed by CICS and RSE Sheffield.
* You will have priority access to your own nodes but when not in use these will be available within the general ShARC pool (this is different to exclusive access).

### Available Software on HPC Machines ###

GPU supported software and libraries are documented on the [ICEBERG GPU Documents](http://rcg.group.shef.ac.uk/iceberg/gpu/index.html) and ShARC (website coming soon) documentation sites. These are community driven sites so if you have documentation on installation of software then please consider [contributing](https://github.com/rcgsheffield/iceberg_software) to this site.

### Software support and Requests ###

If you would like to request new software, libraries or library updates to run GPU code on Iceberg or ShARC then submit an [issue on the github site](https://github.com/rcgsheffield/iceberg_software/issues).

## Contact ##

For queries regarding GPUComputing@Sheffield please contact [Dr Paul Richmond](http://paulrichmond.shef.ac.uk/contact/)
