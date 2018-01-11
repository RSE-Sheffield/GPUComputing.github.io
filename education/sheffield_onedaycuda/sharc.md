---
layout: page
title: Getting Started
permalink: /education/sheffield_onedaycuda/sharc/
---

# GPU Computing Workshop: Getting Started (Day 1) #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

This training course will take place on the University of Sheffield’s new High Performance computing system (ShARC). ShARC has a number of GPU resources available including two nodes with 4x Tesla K80 GPUs (8 devices). This is matched by two additional private K80 nodes and a DGX-1 system with 8x Tesla P100 GPUs. For this course, we will have access to the private nodes to avoid any job queues. Details of how to use these will be given in this document.

*Note: If you wish to develop and execute the exercise from this course locally then this is also possible. The Diamond High spec PC lab has NVIDIA Quadro K5200 GPUs. For instructions on how to develop CUDA code with windows, see the lecture slides titled “CUDA Compilation and execution in Visual Studio”. You will also need to install Visual Studio 2017 (version 15.2) and CUDA 9.1 (in that order) from the CICS software centre.*

## Accessing ShARC GPUS and Initial Setup ##

The ShARC is accessed through an `ssh` terminal. To login to ShARC from a CICS managed desktop account you can use the PuTTY program (if it is unavailable then you will need to install it via the software centre). Within Putty, set the *“Host Name”* text field to `sharc.shef.ac.uk`. For the later labs we will require X windows forwarding. To enable this, ensure the *“Enable X Window Forwarding”* option is selected from the `Connection -> SSH -> X11` tab (see figure below). Select Open and login with your CICS username and password.

{: .center}
![Putty configuration](\static\img\cuda\putty_config.png)

If you are connecting from a machine which is your own (i.e. not a CICS managed desktop) then connect to ShARC using the instructions provided on the [ShARC documentation site](http://docs.hpc.shef.ac.uk/en/latest/sharc/index.html):

[http://docs.hpc.shef.ac.uk/en/latest/hpc/getting-started.html](http://docs.hpc.shef.ac.uk/en/latest/hpc/getting-started.html).

Once you are logged into a ShARC head node then request an interactive session by typing `qrshx`. This creates an interactive session on a CPU worker node which supports running graphical applications as well as command line programs. The worker node **will not** be a GPU accelerated node and **cannot** be used to execute CUDA applications. The worker node can however be used for CUDA compilation, interactive editing of our GPU code and submission of jobs to a GPU accelerated worker. This is a much better solution than running an interactive session on a GPU node as interactive jobs will request GPU resource which will be unused if you are performing tasks like modifying or building your code.

## Configuring the CPU worker node ##

To be able to compile CUDA code within your interactive session on the CPU worker node, we will need to add a couple of modules to the environment. Load the latest version of CUDA using the following command:

	module load libs/CUDA

To compile CUDA programs, you also need a compatible version of the GCC compiler suite. Load version 4.9.4 using the following command:

	module load dev/gcc/4.9.4

*Note: Newer versions of GCC are supported on ShARC but may not work correctly with CUDA. Further notes on the modules are available on the ShARC [software documentation site](http://docs.hpc.shef.ac.uk/en/latest/sharc/software/libs/cuda.html#cuda-sharc).*

Test your CUDA module is correctly loaded by calling:

	nvcc --version

You should get the version information similar to the output below.

	nvcc: NVIDIA (R) Cuda compiler driver
	Copyright (c) 2005-2015 NVIDIA Corporation
	Built on ...
	Cuda compilation tools, release 9.X, VX.X.XX

## Hello World for GPUs ##

From your interactive session on the CPU worker node get the *"hello world for GPUs"* code from Github by cloning the master branch of the `CUDAHelloWorld` repository from the RSE-Sheffield github account. E.g.

	git clone https://github.com/RSE-Sheffield/CUDAHelloWorld.git

This will check out the hello world example into the folder `CUDAHelloWorld`. Take a look at the contents of this file in the console using `nano` e.g.

	nano helloworld.cu

*Note: If you wish to view/edit this file locally then you can use sftp (e.g. using Filezilla or MoboXTerm).*

Compile the code using nvcc with the following command:

	nvcc helloworld.cu –o helloworld

Assuming there are no errors, your code will be built and you will have a new executable file in the working directory.

## Running a Simple Example as a Job ##

We cannot run our compiled GPU code on the worked node as it has no GPUs or CUDA driver. Instead we must submit the execution of the GPU accelerated CUDA program via the ShARC job submission system (`qsub`). The normal command for doing this would be (Note: don’t run the following, it is for reference only).

	qsub -l gpu=1 -b y ./helloworld

The `–l` command allow us to request specific resources in this case a GPU (`gpu=1`). The `–b y` command specifies that we are submitting a binary file rather than a batch script. To avoid queuing on the main ShARC GPU nodes we can use the following extended version of `qsub` for this training session:

	qsub -P rse-training -q rse-training.q -l gpu=1 -b y ./helloworld

This will run the jobs on a private job queue (`-q rse-training.q`) which requires that your CICS username is a member of the training project (`-P rse-training`). All participants of the course have been made members of this group for the duration of the training course (see and instructor if you have a permission error). You will be notified that your job has been submitted but you will not see the output of the executable in the terminal.

Rather than using this long `qsub` command each time you want to run this example, you can instead place the job submission options in a bash script file and submit this. Examine the file `helloworld.sh` which matches the above configurations and submit the job using the following command instead:

	qsub helloworld.sh

The labs in this training course will provide submission batch scripts for you.

## Monitoring the Progress and Output of Your Job ##

To monitor your job, you can use the `qstat` command. i.e.

	qstat –u <your_cics_username>

This will output a table with your jobs. E.g.

	job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
	-----------------------------------------------------------------------------------------------------------------
	205629 0.00534 bash       co1pr        r     01/18/2017 14:24:55 interactive.q@testnode01.icebe     1
	205650 0.00000 helloworld co1pr        qw    01/18/2017 14:26:45                                    1

The status of your GPU job will initially be `qw` to indicate queued and waiting. It will then change to `r` whilst running and will disappear once complete. The job submission will create an execution log file with a file name batching the binary or bash script and a  file extension of `.e<jobid>` (i.e. `helloworld.e205650`). You can view the contents of this log file using the `cat` command. E.g.

	cat helloworld.e205650

If the execution produced no errors then this file will be empty. Once the job is completed a matching output log will be created with a file extension of `*.o<jobid>` (i.e. `helloworld.sh.o205650`). The contents of which should look like the following:

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

---

&#124; [CUDA Training Home](../) &#124; [Lab01](../lab01) &#124;
