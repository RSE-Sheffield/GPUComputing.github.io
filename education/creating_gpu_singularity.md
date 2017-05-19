---
layout: page
title: Creating Portable GPU-Enabled Singularity Images
permalink: /education/creating_gpu_singularity/
---

# Creating Portable GPU-Enabled Singularity Images #

This tutorial will walk you through the creation of a Nvidia GPU-enabled Singularity image that is able to run across host machines with various graphics driver versions.

## Singularity and GPUs ##

[Singularity](http://singularity.lbl.gov/) is a containerization technology similar to Docker. Each Singularity image is a self-contained executable package of a piece of software and everything needed to run it. As Singularity does not require a root-level daemon to run its images, unlike Docker, it is the preferred container technology for use on HPC systems including the ShARC cluster at the University of Sheffield.

The use of Singularity means that an image created on your local development machine that has a complex set of software and dependencies will also work when transferred to another machine, e.g. a HPC cluster, that has Singularity installed.

The problem faced when trying to run a Singularity image with the software that uses Nvidia GPUs (e.g. CUDA) is that driver files must be present and the version must match the one installed on the host system. Embedding of driver files within the image creates non-portable container that only works on a single driver version. Fortunately it is possible to host the driver files externally and mount them in to the image at run-time.

## Installing Singularity on your Machine ##

First, you will need Singularity installed on your machine in order to locally run, create and modify images. The following is the installation command for Debian/Ubuntu based systems:

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

Many pre-built images are available from the [Singularity hub](https://singularity-hub.org). Additionally, Singularity has the ability to create images from Docker containers pulled directly from [Docker hub](https://hub.docker.com/). For this tutorial, we will be creating an image based on the [Nvidia CUDA docker container](https://hub.docker.com/r/nvidia/cuda/).

First, an empty image must be created. The following command creates an image named `cuda.img` of the size 2500 MB:

```

sudo singularity create -s 2500 cuda.img

```

We then create a Singularity definition file which is a recommended way for making reproducible Singularity images. Create a file called `cuda.def` with the contents below:

```
Bootstrap: docker
From: nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04

%setup
  #Runs on host. The path to the image is $SINGULARITY_ROOTFS

%post
  #Post setup, runs inside the image

  #Default mount point used in Shef Uni's ShARC cluster
  mkdir /scratch /data /shared /fastdata

  #Nvidia driver file mount paths
  mkdir /nvlib /nvbin

  #Add nvidia driver paths to the environment variables
  echo "\n #Nvidia driver paths \n" >> /environment
  echo 'export PATH="/nvbin:$PATH"' >> /environment
  echo 'export LD_LIBRARY_PATH="/nvlib:$LD_LIBRARY_PATH"' >> /environment

  #Add CUDA paths
  echo "\n #Cuda paths \n" >> /environment
  echo 'export CPATH="/usr/local/cuda/include:$CPATH"' >> /environment
  echo 'export PATH="/usr/local/cuda/bin:$PATH"' >> /environment
  echo 'export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"' >> /environment
  echo 'export CUDA_HOME="/usr/local/cuda"' >> /environment

%runscript
  #Executes when the "singularity run" command is used
  #Useful when you want the container to run as an executable


%test
  #Test script to verify that the image is built and running correctly

```

The definition file takes a base image from [docker hub](https://hub.docker.com/) (`Bootstrap: docker`). In this case we're using the image that comes with Ubuntu 16.04, CUDA 8.0 and cuDNN 5.0 pre-installed (`From nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04`).

In the `%post` section of the definition file, we created two directories at the root, `/nvbin` for the driver executables like `nvidia-smi`  and `/nvlib` for the driver library files. These directories are added to the `PATH` and `LD_LIBRARY_PATH` environment variables by appending it to the `/environment` file in the image which is equivalent to the `.bashrc` file in `bash`.

When creating images from the Docker hub, some environment variables may be missing. As can be seen in our definition script we've had to add the CUDA paths (e.g. `/usr/local/cuda/bin` and `/usr/local/cuda/lib64` ) to the `/environment` file manually.

Having created the definition file, use the ``bootstrap`` command to build the image you've just created:

```

sudo singularity bootstrap cuda.img cuda.def

```

You've just created and bootstrapped a Singularity image that is ready to use CUDA once we've provided it with the correct driver files.

## Getting the Nvidia Driver files for Your Host Machine ##

**This process must be done for each host machine you're using to run the GPU-enabled Singularity images.**

Use the following command to find your current driver version:

```

nvidia-smi

```

You will get something similar to the following:

```

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

```

It can be seen that the driver version on our current machine is ``367.57``. Go to the [Nvidia website](http://nvidia.com) and search for the correct Linux driver installer (`.run` file) for your graphics card.

Create a file called `extractnvdriver.sh` with the following code:

```
#!/bin/bash

NVID_VER=$1
DEST_DIR=$2

if [ "$#" -ne 2 ]; then
    echo "Use as follows: \n extract_driver_and_moveto.sh version_number path/to/move \n example: \n extractnvdriver.sh 367.57 ~/nvdriver"
    exit 1
fi

if [ ! -f "NVIDIA-Linux-x86_64-${NVID_VER}.run" ]; then
    echo "Driver installer file NVIDIA-Linux-x86_64-${NVID_VER}.run not found"
    exit 1
fi

if mkdir -p $DEST_DIR ; then
    echo "Created directory at $DEST_DIR"
else
    echo "Could not create directory at $DEST_DIR"
    exit 1
fi

chmod 755 ./NVIDIA-Linux-x86_64-${NVID_VER}.run

# extract nvidia files
if ./NVIDIA-Linux-x86_64-${NVID_VER}.run --extract-only ; then
  echo "Extracted driver"
else
  echo "Could not extract driver"
  exit 1
fi

# make links in nvidia directory
cd NVIDIA-Linux-x86_64-${NVID_VER}
ln -s libGL.so.${NVID_VER}         libGL.so.1
ln -s libEGL_nvidia.so.${NVID_VER}         libEGL_nvidia.so.0
ln -s libGLESv1_CM_nvidia.so.${NVID_VER}   libGLESv1_CM_nvidia.so.1
ln -s libGLESv2_nvidia.so.${NVID_VER}      libGLESv2_nvidia.so.2
ln -s libGLX_nvidia.so.${NVID_VER}         libGLX_indirect.so.0
ln -s libGLX_nvidia.so.${NVID_VER}         libGLX_nvidia.so.0
ln -s libnvidia-cfg.so.1                   libnvidia-cfg.so
ln -s libnvidia-cfg.so.${NVID_VER}         libnvidia-cfg.so.1
ln -s libnvidia-encode.so.1                libnvidia-encode.so
ln -s libnvidia-encode.so.${NVID_VER}      libnvidia-encode.so.1
ln -s libnvidia-fbc.so.1                   libnvidia-fbc.so
ln -s libnvidia-fbc.so.${NVID_VER}         libnvidia-fbc.so.1
ln -s libnvidia-ifr.so.1                   libnvidia-ifr.so
ln -s libnvidia-ifr.so.${NVID_VER}         libnvidia-ifr.so.1
ln -s libnvidia-ml.so.1                    libnvidia-ml.so
ln -s libnvidia-ml.so.${NVID_VER}          libnvidia-ml.so.1
ln -s libnvidia-opencl.so.${NVID_VER}      libnvidia-opencl.so.1
ln -s vdpau/libvdpau_nvidia.so.${NVID_VER} libvdpau_nvidia.so
ln -s libcuda.so.${NVID_VER}               libcuda.so
ln -s libcuda.so.${NVID_VER}               libcuda.so.1

# move into place (overwrite old if exist)
cd ..
if mv NVIDIA-Linux-x86_64-${NVID_VER}/* $DEST_DIR ; then
  echo "Successfully installed to $DEST_DIR"
else
  echo "Cannot install drivers to $DEST_DIR"
  exit 1
fi

```

Then run the script like below in the same directory as the Nvidia driver installer you downloaded. **Remember to replace the number `367.57` with the driver version that you're using.** :

```

chmod +x extract_nvdriver_and_moveto.sh
extractnvdriver.sh 367.57 ~/mynvdriver

```

If the extraction is successful, the driver files will be located at `~/mynvdriver`. We're now ready to test the image that we've just built.

*Note that `~/mynvdriver` directory contains both the executable and library files and so in this case `/nvbin` and `/nvlib` will actually be mounted to the same location. Instead of downloading the drivers, it is possible to use the existing driver files on your machine instead if you already know their location, in this case the executable and library files are often located in different folders. Ensure that there are ONLY Nvidia driver executables and libraries in the folder in order to prevent contamination.*

*The driver extraction code is taken from a part of the [gpu4singularity script by NIH](https://hpc.nih.gov/apps/singularity.html).*

## Running and testing the Image ##

To get a shell in to the image, go to the directory that has your `cuda.img` and run the following command:

```

singularity shell -B ~/mynvdriver:/nvlib,~/mynvdriver:/nvbin myimage.img

```

The `-B ~/mynvdriver:/nvlib,~/mynvdriver:/nvbin` flag tells singularity to mount the `~/mynvdriver` to the `/nvlib` directory inside the image, and same with `/nvbin`.

A good first test is to run the `nvidia-smi` command from inside the image:

```

nvidia-smi

```

And you should see the same result as when you excuted the command on the host machine before:

```

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

```

Let's try compiling and running a test cuda code. Create a file called `test.cu` with the contents:

```

#include <stdio.h>
#include <cuda.h>

// Kernel that executes on the CUDA device
__global__ void square_array(float *a, int N)
{
  int idx = blockIdx.x * blockDim.x + threadIdx.x;
  if (idx<N) a[idx] = a[idx] * a[idx];
}

// main routine that executes on the host
int main(void)
{
  float *a_h, *a_d;  // Pointer to host & device arrays
  const int N = 10;  // Number of elements in arrays
  size_t size = N * sizeof(float);
  a_h = (float *)malloc(size);        // Allocate array on host
  cudaMalloc((void **) &a_d, size);   // Allocate array on device
  // Initialize host array and copy it to CUDA device
  for (int i=0; i<N; i++) a_h[i] = (float)i;
  cudaMemcpy(a_d, a_h, size, cudaMemcpyHostToDevice);
  // Do calculation on device:
  int block_size = 4;
  int n_blocks = N/block_size + (N%block_size == 0 ? 0:1);
  square_array <<< n_blocks, block_size >>> (a_d, N);
  // Retrieve result from device and store it in host array
  cudaMemcpy(a_h, a_d, sizeof(float)*N, cudaMemcpyDeviceToHost);
  // Print results
  for (int i=0; i<N; i++) printf("%d %f\n", i, a_h[i]);
  // Cleanup
  free(a_h); cudaFree(a_d);
}

```

Compile with `nvcc` then run the code:

```

nvcc test.cu -o test
./test

```

If successful you should see the following results:

```

0 0.000000
1 1.000000
2 2.000000
3 3.000000
4 4.000000
5 5.000000
6 6.000000
7 7.000000
8 8.000000
9 9.000000

```


## Auto-mounting GPU Driver files using the Singularity Configuration File ##

Having extracted the driver files, you can use the Singularity Configuration file `singularity.conf` to automatically mount the driver files without the need to use the `-B` flag. If you installed Singularity using the instructions above, the config file should be located at `/usr/local/etc/singularity/singularity.conf`.

Add the following two lines to your `singularity.conf` to have the driver files automatically mount to your image:

```

bind path = ~/mynvdriver:/nvbin
bind path = ~/mynvdriver:/nvlib

```

Now when you run GPU-dependent program in the image without the `-B` flag, it should execute without error e.g.:

```

singularity exec myimage.img nvidia-smi

```

## Problems & Feedbacks ##

If you encounter any problems or have feedbacks on how to improve this tutorial, please get in touch with the [RSE Sheffield Group](http://rse.shef.ac.uk/contact/).
