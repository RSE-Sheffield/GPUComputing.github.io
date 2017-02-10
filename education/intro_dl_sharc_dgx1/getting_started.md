---
layout: page
title: Getting Started
permalink: /education/intro_dl_sharc_dgx1/getting_started/
---

# Introduction to Deep Learning on ShARC's DGX-1: Getting Started #


This training course will take place on the University of Sheffield’s new High Performance computing system (ShARC). ShARC has a number of GPU resources available including two nodes with 4x Tesla K80 GPUs (8 devices). This is matched by two additional private K80 nodes and a DGX-1 system with 8x Tesla P100 GPUs. For this course, we will have exclusive access to the DGX-1 to avoid any job queues. Details of how to use these will be given in this document.

## Accessing ShARC and Initial Setup ##

The ShARC is accessed through an `ssh` terminal. To login to ShARC from a CICS managed desktop account you can use the PuTTY program (if it is unavailable then you will need to install it via the software centre). Within Putty, set the *“Host Name”* text field to `sharc.shef.ac.uk`. For the later labs we will require X windows forwarding. To enable this, ensure the *“Enable X Window Forwarding”* option is selected from the `Connection -> SSH -> X11` tab (see figure below). Select Open and login with your CICS username and password.

{: .center}
![Putty configuration](\static\img\cuda\putty_config.png)

On a Linux machine, simply call `ssh` from the terminal and type in your CiCs password when prompted.

```
	ssh -X [your cics username]@sharc.shef.ac.uk
```

If you are connecting from a machine which is your own (i.e. not a CICS managed desktop) then connect to ShARC using the instructions provided on the ShARC documentation site:

[http://docs.hpc.shef.ac.uk/en/latest/hpc/getting-started.html](http://docs.hpc.shef.ac.uk/en/latest/hpc/getting-started.html).

Once you are logged into a ShARC head node then request an interactive session by typing `qrshx`. This creates an interactive session on a CPU worker node which supports running graphical applications as well as command line programs. The worker node **will not** be a GPU accelerated node and and it is advised that you **not** run any Neural Network training on it. The worker node can however be used for interactive editing of our deep learning models and submission of jobs to the DGX-1 or other GPU-enabled nodes. This is a much better solution than running an interactive session on a GPU node as interactive jobs will request GPU resource which will be unused if you are performing tasks like modifying or building your code.






ShARC uses a similar system to Iceberg so commands for submitting jobs (`qsub`) and requesting interactive sessions (`qrsh, qsh, qrshx` ) still applies. See the Iceberg [documentation](http://docs.iceberg.shef.ac.uk/en/sharc/hpc/index.html) for further details.

ShARC is located at `sharc.shef.ac.uk` so to SSH from a linux system:



The `-X` option allows for X11 forwarding. This may be necessary if you're planning to use GUI applications e.g. Matlab.



### Submitting jobs to the DGX-1

To make efficient use of DGX-1's computing resource we recommend submitting batch jobs instead of using interactive session.

First create a job script named `my_script.sh` with the following content

```
	#!/bin/bash
	#$ -l gpu=1 -P rse -q rse-training.q -l rmem=10G -l h_rt=00:20:00

	echo "Hello world"
```

In the second line of the script `-l gpu=1 -P rse -q rse.q` request the jobs to be added to the RSE queue that has the DGX-1, `-l gpu=1` specify that you want one GPU. Other options can be added such as `-l rmem=10G` asks for 10GB of 'real' memory or `-l h_rt=00:20:00` to indicate how long the jobs can  run for (default is 8 hours), see the [documentation](https://www.shef.ac.uk/cics/research/hpc/sharc/batch) for the full list.

Make your script executable and run it with the `qsub` command

```
	chmod +x my_script.sh
	qsub my_script.sh
```

Check your script status with `qstat` command

```
	qstat
```

Which will show you something similar to

```
job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
-----------------------------------------------------------------------------------------------------------------
   4555 0.00023 bash       ac1tk        r     01/26/2017 15:14:26 gpu.q@sharc-node100.shef.ac.uk     1
   4570 0.31087 my_script. ac1tk        r     01/26/2017 15:24:57 all.q@sharc-node083.shef.ac.uk     1
```

The `state` column shows the current status of the job, `q` means queueing and `r` means running.

Two files are created in the folder that you ran `qsub`, one for standard output `my_script.sh.o4570` and one for error/warning `my_script.sh.e4570`. Let's see the contents with the `cat` command

```
cat my_script.sh.o4570
```

You'll get

```
	Hello world
```

You can also put the parameters on the `qsub` command rather than in the script e.g.

```
	qsub -l gpu=1 -P rse -q rse-training.q my_script.sh
```

Other useful options include

Combining the output and error in to a single file:

```
	-j y
```

Specifying location and name of your our output log:

```
	#For output file
	-o your/path

	#For error file, don't add this if you've used the -j option
	-e your/path
```

Sending you an e-mail when the job is completed/aborted/suspended:

```
 	-M your.email@yourdomain.com -m eas
```



### Getting your environment ready for Caffe

There are two version of Caffe on ShARC (see [documentation](https://github.com/RSE-Sheffield/GPUComputing/blob/master/deeplearning/Caffe.rst)), the default version which is more updated and Nvidia's version that is more optimised for the GPU. We'll use the default version as it has support for recurrent networks.

To use the python bindings for Caffe we need to prepare our python environment. First we load up the Caffe module which also loads Conda, CUDA, cuDNN and GCC

```
	module load libs/caffe/rc3/gcc-4.9.4-cuda-8.0-cudnn-5.1-conda-3.4-TESTING
```

We'll create a local python 3.5 environment named `caffe`, activate it then install `matplotlib` and `numpy`

```
	conda create -n caffe python=3.5
	source activate caffe
	conda install -y matplotlib numpy scikit-image
```

Every time you log in to the node and in all your job scripts, you need to load the module and activate the caffe conda environment again

```
	module load libs/caffe/rc3/gcc-4.9.4-cuda-8.0-cudnn-5.1-conda-3.4-TESTING
	source activate caffe
```

### Downloading the code for the practicals

Run

```
	git clone https://github.com/RSE-Sheffield/DLTraining.git
```

Now we're ready to start the first practical.
