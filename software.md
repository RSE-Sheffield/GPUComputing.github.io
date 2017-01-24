---
layout: default
title: GPU Accelerated Software
permalink: /software/
---

# Software #

Utilisation of GPUs (and other accelerator architectures) is provided through software which varies in the level of complexity. The GPUs architecture is reasonably complicated and programming the device efficiently is mixture of skill and art. It is not however required to understand the architecture to use software which has been accelerated. Many software libraries have very efficient GPU back-ends which are abstracted from the end user behind some form of domain specific language or API. 

## Programming GPUs ##


### CUDA ###

There are various options for programming GPUs. The most commonly used within research in the NVIDIA CUDA C programming language (which as the name suggests, is only available for NVIDIA GPUS). CUDA is a low-level language which requires that you understand the GPUs architecture to achieve optimal performance. It has a both a runtime and driver API. The runtime API is significantly more user friendly is what most people use for CUDA programming. If you can not find a library or piece of software that fulfils your GPU computing need or want to write your own library then you will most likely use CUDA.

### OpenCL ###

OpenCL is a cross platform C API for programming GPUs and multi-core CPUs. Unlike CUDA it is able to target both NVIDIA and ATI GPUs as well as Intel multi-core CPUs. OpenCL lacks some of the latest functionality of CUDA (which is specific to NVIDIA GPUs) and hence has a less active user community, especially within research. The API is lower-level than CUDA and is more similar to the CUDA driver API which requires manually loading and unloading of code to the GPU device. 

### OpenACC ###

OpenACC is a compiler extension by the Portland compiler group which allows standard C or Fortran code to be compiled for GPUs. This is done inn a similar way to OpenMP by adding compiler directives around for loops. This method of parallelisation is easy to use and can quickly add GPU acceleration to existing code. Compiler directives are every efficient at abstracting the GPU architecture from users but this can be both a blessing and a curse. A well optimised CUDA implementation will likely out perform an OpenACC implementation.

### Thrust ###

Thrust is a C++ header library API which aims to allow uses to add GPU acceleration to their code. Thrust is essentially a GPU implementation of many common algorithms from the STL. The implementation of algorithms such as sort, search, prefix-sum and reduction are provided in efficient templated library functions refered to as parallel primitives. It is ofern possible to combine various primates together to achieve a new algorithm without having to write this in CUDA. Thrust algorithms can however be combined with CUDA code making the library very powerful.


## Deep Learning Software ##

Deep Learning (DL) on GPUs has led to what NVIDA describe as the "AI era of computing". DL is the application of (deep) neural networks for machine learning and GPUs are ideally suited to the computationally expensive task of training these networks with large amounts fo data. The importance of DL to GPU computing can not be understated and huge investments in DL software have been made. Of particular interest are TensorFlow, Theano, Caffe and Torch, each of which presents a high level syntax for describing DL network architectures and APIs for training and inference (the application of a trained network to new data). Whilst easy to use, initial installation and configuration of these pieces of software is notoriously difficult, especially on advanced machine configurations (e.g. with multiple GPUs). It is recommended to use containerisation (e.g. docker or conda) on your won machines or use software which has been pre-configured for the University of Sheffield HPC systems.


## Accelerated Libraries ##

Accelerated libraries abstract the GPU from users completely which will typically use a domain specific language of API which the library efficiently translates to GPU code (usually with a CUDA backend). The DL software is an example of an accelerated library. Other examples include Matlab (which ahs GPU computing support for certain libraries), cuBLAST for CUDA accelerated BLAS, cuSPARSE for CUDA accelerated sparse matrices) and many others.


## Software Support at University of Sheffield ##

GPU supported software and libraries upported on the universities HPC systems is documented on the [HPC Documents Wesbite](http://docs.hpc.shef.ac.uk/). These are community driven sites so if you have documentation on installation of software then please consider [contributing](https://github.com/rcgsheffield/iceberg_software) to this site. The [Sheffield RSE community](http://www.rse.shef.ac.uk) frequently add software to this site to support University of Sheffield researchers and software developers.

If you would like to request new software, libraries or library updates to run GPU code on Iceberg or ShARC then submit an [issue on the github site](https://github.com/rcgsheffield/iceberg_software/issues). If you require longer term support then consider costing dedicated RSE support into your next research grant. [Contact RSE Sheffield](http://www.rse.shef.ac.uk/contact) for details.
 
