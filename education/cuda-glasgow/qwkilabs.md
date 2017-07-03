---
layout: page
title: Getting Started
permalink: /education/cuda-glasgow/qwiklabs/
---

# GPU Computing Workshop: Getting Started (Day 1) #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

This training course will take place using the [NVIDIA Qwiklabs](https://nvlabs.qwiklab.com/) system. This is a cloud based training portal for GPU computing which uses Amazon AWS with GPU images. In order to access a GPU accelerated instance go to [NVIDIA quiklabs](https://nvlabs.qwiklab.com/) and create a free student account (Important: You must use the email address you provided when registering). Once created, you should be able to select the 'Introduction to CUDA and DL Glasgow' course (we will use this for both the CUDA and Deep Learning parts of the workshop) and then select `Start Lab`. After ~7 minutes the instance will start and you will be provided with connection information.

## Logging in to you Amazon Instance ##

The connection information will provide you with an Amazon cloud instance address and username password which can be accessed via an `ssh` terminal. To login from linux you can use the following command (note the requirement of -X for graphical window forwarding);

	ssh -X ubuntu@10-11-12-13.amazon.eu....

If you are in MacOSX then you can use the same command as above but you will need to install the X server ([more](https://support.apple.com/en-gb/HT201341)).

From Windows you can use the PuTTY program (if it is unavailable then you will need to install it). You will also need to install an X server. The free XMing application is recommended ([XMing](https://sourceforge.net/projects/xming/)). Within Putty, set the *“Host Name”* text field to your amazon instance e.g. `ubuntu@10-11-12-13.amazon.eu....`. To enable X windows forwarding ensure the *“Enable X Window Forwarding”* option is selected from the `Connection -> SSH -> X11` tab (see figure below). Select Open and login with the provided username and password.

{: .center}
![Putty configuration](\static\img\cuda\putty_config.png)

To be able to compile CUDA code within your interactive session on the CPU worker node, we will be usin the NVIDIA CUDA compiler (`nvcc`). Confirm this is installed by calling;

	nvcc --version

You should get the version information similar to the output below.

	nvcc: NVIDIA (R) Cuda compiler driver
	Copyright (c) 2005-2015 NVIDIA Corporation
	Built on Tue_Aug_11_14:49:10_CDT_2015
	Cuda compilation tools, release 7.5, V7.5.17

## Hello World for GPUs ##

Get the *"hello world for GPUs"* code from Github by cloning the master branch of the `CUDAHelloWorld` repository from the RSE-Sheffield github account. E.g.

	git clone https://github.com/RSE-Sheffield/CUDAHelloWorld.git

This will check out the hello world example into the folder `CUDAHelloWorld`. You can ignore the `helloworld.sh` file which is used for launching the executable on compute clusers. Take a look at the contents of the cuda source file in the console using `nano` e.g.

	nano helloworld.cu

*Note: If you wish to view/edit this file locally then you can use sftp (e.g. using Filezilla or MoboXTerm).*

Compile the code using nvcc with the following command:

	nvcc helloworld.cu –o helloworld

Assuming there are no errors, your code will be built and you will have a new executable file in the working directory. Execute the compiled program;

	./helloworld

The outputs should look like the following:

	Hello World from Thread 0
	Hello World from Thread 1
	Hello World from Thread 2
	Hello World from Thread 3
	Hello World from Thread 4
	Hello World from Thread 5
	Hello World from Thread 6
	Hello World from Thread 7
	Hello World from Thread 8
	Hello World from Thread 9

## Modifying the Hello Word Example ##

Before moving onto the [first lab classes](../lab01) for day 1, try modifying the grid and block dimensions to see how the thread index changes. Try using more than one block and add the block index (from `blockIdx.x`) to the `printf` statement. If you want to use 2D or 3D blocks then use a `dim3` variable to define the grid and block size. E.g.

	dim3 blocksPerGrid(2, 2, 1);
	dim3 threadsPerBlock(3,3, 1);

You can now launch your kernel using these dim variables.

	hello_kernel<<<blocksPerGrid, threadsPerBlock>>>();

To view the y and z index of the thread or the block use the y and z member variables of the `threadIdx` or `blockIdx` dims.

---

&#124; [CUDA Training Home](../) &#124; [Lab01](../lab01) &#124;
