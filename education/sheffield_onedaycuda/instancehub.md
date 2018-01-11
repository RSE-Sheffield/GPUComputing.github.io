---
layout: page
title: Getting Started
permalink: /education/sheffield_onedaycuda/instancehub/
---

# Cloud GPU Instance with InstanceHub #

InstanceHub is a new website (you will be one of the first people to test it) which allows a trainer to provide access to AWS EC2 cloud instances. I have configured a lab which will provide you with a GPU enabled EC2 instance with the software and lab classes pre-installed. To access your instance follow the link below and create an account using the email address you used to register for the course.

[https://www.instancehub.com/labs/3/](https://www.instancehub.com/labs/3/)

A description on how to use the instance is provided on the website. You can either SSH directly or use the provided Python Notebook server to create a web based terminal and edit your source files. The remaining instructions assume you are using the notebook server but if you wish to use SSH then feel free.

## Hello World for GPUs ##

If you launch the Python notebook server from the lab description then you can create a terminal session by selecting `New -> Terminal` from the Notebook Menu.

You can navigate the files within the notebook from the notebook server. Open the HelloWorldCUDA folder and select the `helloworld.cu` file to edit it in the browser.

Open the terminal browser window you created, navigate to the `HelloWorldCUDA` directory and compile the code using nvcc with the following command:

	nvcc helloworld.cu -o helloworld

Assuming there are no errors, your code will be built and you will have a new executable file in the working directory.

## Running a Simple Example as a Job ##

There is no need to use job submission on the EC2 instance you can run the compiled cuda executables directly in the terminal. I.e.

	./helloworld
	
This should produce the following output

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

	dim3 grid(2, 2, 1);
	dim3 block(3,3, 1);

You can now launch your kernel using these dim variables.

	helloworld<<<grid, block>>>();

To view the y and z index of the thread or the block use the y and z member variables of the `threadIdx` or `blockIdx` dims.

For the remaining labs you will not need to check the code out as instructed as can work on it and execute it directly as you have for the `helloworld` example. 

Not: To view the image outputs for lab2 you will need to select the files from the notebook server and download them as opening them will assume you want to edit them as text.

---

&#124; [CUDA Training Home](../) &#124; [Lab01](../lab01) &#124;

