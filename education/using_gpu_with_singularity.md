---
layout: page
title: GPU-enabled Singularity
permalink: /education/gpu_singularity/
---

# GPU-enabled Singularity #

This tutorial will walk you through the creation of a Nvidia GPU-enabled Singularity image that is able to run across host machines with various graphics driver versions.

## Singularity and GPUs ##

[Singularity](http://singularity.lbl.gov/) is a containerization technology similar to Docker. Each Singularity image is a self-contained executable package of a piece of software and everything needed to run it. As Singularity does not require a root-level daemon to run its images, unlike Docker, it is the preferred container technology for use on HPC systems including the ShARC cluster at the University of Sheffield.

This in-practice means that an image created on your local development machine that has a complex set of software and dependencies will also work when uploaded to a HPC cluster that supports Singularity.

The problem faced when trying to run a Singularity image with the software that uses Nvidia GPUs (e.g. CUDA) is that driver files must be present and the version must match the one installed on the host system. Embedding of driver files within the image creates non-portable container that only works on a single driver version. Fortunately it is possible to host the driver files externally and mount them in to the image at run time.

## Installing Singularity on your Machine ##

First you will need Singularity installed on your machine in order to locally run, create and modify images. The following is the installation command for debian/ubuntu based systems:

```
  sudo apt-get update
  sudo apt-get -y install build-essential curl git sudo man vim autoconf libtool automake
  git clone https://github.com/singularityware/singularity.git
  cd singularity
  ./autogen.sh
  ./configure --prefix=/usr/local
  make
  sudo make install
```

For other linux distributions, see the official documentation on the [Singularity website](http://singularity.lbl.gov/).

## Creating an image that uses the GPU ##

Firstly an empty image must be created. The following command creates an image named ``myimage.img`` of the size 1024 MB: ::

  sudo singularity create -s 1024 myimage.img

Singularity uses a definition file for bootstrapping an image. An example definition ``ShARC-Ubuntu-Base.def`` is shown below ::

  Bootstrap: docker
  From: ubuntu:latest

  %setup
  	#Runs on host. The path to the image is $SINGULARITY_ROOTFS

  %post
  	#Post setup, runs inside the image

    #Default mount paths
  	mkdir /scratch /data /shared /fastdata

    #Nvidia driver mount paths, only needed if using GPU
  	mkdir /nvlib /nvbin

    #Add nvidia driver paths to the environment variables
  	echo "\n #Nvidia driver paths \n" >> /environment
  	echo 'export PATH="/nvbin:$PATH"' >> /environment
  	echo 'export LD_LIBRARY_PATH="/nvlib:$LD_LIBRARY_PATH"' >> /environment

  %runscript
    #Runs inside the image every time it starts up

  %test
    #Test script to verify that the image is built and running correctly

The definition file takes a base image from [docker hub](https://hub.docker.com/), in this case the latest version of Ubuntu ``ubuntu:latest``. Other images on the hub can also be used as the base for the Singularity image, e.g. ``From: nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04`` uses Nvidia's docker image with Ubuntu 16.04 that already has CUDA 8 installed.

After creating a definition file, use the ``bootstrap`` command to build the image you've just created: ::

  sudo singularity bootstrap myimage.img ShARC-Ubuntu-Base.def

You can also modify the contents of an image after it's been created using the ``-w`` flag: ::

  sudo singularity shell -w myimage.img

The command above gives you a shell in to the image with root access that can then be used to modify its contents.

## Using Nvidia GPU with Singularity Images on Your Local Machine ##


**Support is only available for machines with Nvdia GPUs and will not work for other GPU manufacturers (e.g. AMD).**

We will mount to /nvbin /nvlib

On the ShARC cluster, these driver files are stored outside of the image and automatically mounted to the folders ``/nvbin`` and ``/nvlib`` at run-time. To use the images locally on your machine you simply need to provide the correct driver files for the machine you're using.

Use the following command to find your current driver version: ::

  nvidia-smi

Where you will get something similar to the following: ::

  Tue Mar 28 16:43:08 2017
  +-----------------------------------------------------------------------------+
  | NVIDIA-SMI 367.57                 Driver Version: 367.57                    |
  |-------------------------------+----------------------+----------------------+
  | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
  | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
  |===============================+======================+======================|
  |   0  GeForce GTX TITAN   Off  | 0000:01:00.0      On |                  N/A |
  | 30%   35C    P8    18W / 250W |    635MiB /  6078MiB |      1%      Default |
  +-------------------------------+----------------------+----------------------+

It can be seen that the driver version on our current machine is ``367.57``. Go to the `Nvidia website <http://nvidia.com>`_ and search for the correct Linux driver for your graphics card. Download the `extract_nvdriver_and_moveto.sh </sharc/software/apps/singularity/extract_nvdriver_and_moveto.sh>` to the same directory and run it like so: ::

  chmod +x extract_nvdriver_and_moveto.sh
  extract_driver_and_moveto.sh 367.57 ~/mynvdriver

If you're using the Singularity definition file as shown above (see :ref:`create_image_singularity_sharc`), the ``/nvbin`` and ``/nvlib`` directories will have been created. They simply need to be correctly mounted when running the image using the command where our extracted driver files are located at ``~/mynvdriver``: ::

  singularity shell -B ~/mynvdriver:/nvlib,~/mynvdriver:/nvbin myimage.img

## Running the Image ##



## Auto-mounting GPU Drivers using Singularity Configuration File ##
