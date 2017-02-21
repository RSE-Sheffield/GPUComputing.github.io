---
layout: page
title: Lab 05 Multi-GPU and Benchmarking
permalink: /education/intro_dl_sharc_dgx1/lab05/
---

# Lab 05: Multi-GPU and Benchmarking #

**Remember to be working from the root directory of DLTraining code sample throughout all practicals.**

In this lab, we will be looking at the use of multiple GPUs for running our models. We will also look at Cafffe's facilities for benchmarking the models for us.

## Getting the GPU topology ##

The topology matrix can be obtained using:

```
nvidia-smi topo -m
```

For the DGX-1 you will get something like:

```
	    GPU0GPU1GPU2GPU3GPU4GPU5GPU6GPU7mlx5_0	mlx5_2	mlx5_1	mlx5_3	CPU Affinity
GPU0	 X 	NV1	NV1	NV1	NV1	SOC	SOC	SOC	PIX	SOC	PHB	SOC	1-1
GPU1	NV1	 X 	NV1	NV1	SOC	NV1	SOC	SOC	PIX	SOC	PHB	SOC	1-1
GPU2	NV1	NV1	 X 	NV1	SOC	SOC	NV1	SOC	PHB	SOC	PIX	SOC	1-1
GPU3	NV1	NV1	NV1	 X 	SOC	SOC	SOC	NV1	PHB	SOC	PIX	SOC	1-1
GPU4	NV1	SOC	SOC	SOC	 X 	NV1	NV1	NV1	SOC	PIX	SOC	PHB	1-1
GPU5	SOC	NV1	SOC	SOC	NV1	 X 	NV1	NV1	SOC	PIX	SOC	PHB	1-1
GPU6	SOC	SOC	NV1	SOC	NV1	NV1	 X 	NV1	SOC	PHB	SOC	PIX	1-1
GPU7	SOC	SOC	SOC	NV1	NV1	NV1	NV1	 X 	SOC	PHB	SOC	PIX	1-1
mlx5_0	PIX	PIX	PHB	PHB	SOC	SOC	SOC	SOC	 X 	SOC	PHB	SOC
mlx5_2	SOC	SOC	SOC	SOC	PIX	PIX	PHB	PHB	SOC	 X 	SOC	PHB
mlx5_1	PHB	PHB	PIX	PIX	SOC	SOC	SOC	SOC	PHB	SOC	 X 	SOC
mlx5_3	SOC	SOC	SOC	SOC	PHB	PHB	PIX	PIX	SOC	PHB	SOC	 X

Legend:

  X   = Self
  SOC  = Connection traversing PCIe as well as the SMP link between CPU sockets(e.g. QPI)
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe switches (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing a single PCIe switch
  NV#  = Connection traversing a bonded set of # NVLinks
```

For best performance, peer to peer (P2P) direct memory access (DMA)  between devices is needed. Look for the GPUs that are connected NVLink (`NV1`). Without P2P access, for example crossing PCIe root complex, data is copied through host and effective exchange bandwidth is greatly reduced.

## Training models on multiple GPUs with Caffe ##

Caffe only supports the data parallel approach for multi-GPU and only for training on the command line (other interfaces coming soon).

In your job scripts you have been using the parameter `-l gpu=1` which means we're requesting 1 GPU. Simply replace the number for the GPUs you require e.g. for 4 GPUs:

```
#!/bin/bash
#$ -l gpu=4 -P rse-training -q rse-training.q -l rmem=10G

module load libs/caffe/rc3/gcc-4.9.4-cuda-8.0-cudnn-5.1-conda-3.4-TESTING
source activate caffe
export LD_LIBRARY_PATH="/home/$USER/.conda/envs/caffe/lib:$LD_LIBRARY_PATH"

#Run caffe training below....
```

*The scheduler will try to block out consecutive GPUs if available but is not gauranteed especially during busy times.*

Let's use multi-gpu to train our previous model to train on GPU 0 and 1:

```
caffe train -solver=code/lab05/mnist_lenet_solver.prototxt -gpu=0,1
```

NOTE: each GPU runs the batchsize specified in the `Data` layer of your model.prototxt. If you go from 1 GPU to 2 GPU, your effective batchsize will double. e.g. if your model.prototxt specified a batchsize of 256, if you run 2 GPUs your effective batch size is now 512. So you need to adjust the batchsize when running multiple GPUs and/or adjust your solver params, specifically learning rate.

The current implementation uses a tree reduction strategy. e.g. if there are 4 GPUs in the system, 0:1, 2:3 will exchange gradients, then 0:2 (top of the tree) will exchange gradients, 0 will calculate updated model, 0->2, and then 0->1, 2->3.

Caffe notes on multi-gpu: [https://github.com/BVLC/caffe/blob/master/docs/multigpu.md](https://github.com/BVLC/caffe/blob/master/docs/multigpu.md)

## Benchmarking your models ##

Caffe provides a convenient tool for benchmarking your model with `caffe time`, use as follows:

```
# (These example calls require you complete the LeNet / MNIST example first.)
# time LeNet training on CPU for 10 iterations
caffe time -model code/lab05/mnist_lenet.prototxt -iterations 10

# time LeNet training on GPU for the default 50 iterations
caffe time -model code/lab05/mnist_lenet.prototxt  -gpu 0

# time a model architecture with the given weights on the first GPU for 10 iterations
caffe time -model code/lab05/mnist_lenet.prototxt  -weights mnist_lenet_iter_10000.caffemodel -gpu 0 -iterations 10

# time a model architecture with the given weights on the first two GPUs for 10 iterations
caffe time -model code/lab05/mnist_lenet.prototxt  -weights mnist_lenet_iter_10000.caffemodel -gpu 0,1 -iterations 10
```

Use the `-gpu` flag to declare which GPU to use. To use GPU 0 and 1 for example use the flag `-gpu 0,1` or to use all GPUs you can set `-gpu all`.



---

&#124; [Home](../) &#124; [Lab04](../lab04) &#124; [Lab06](../lab06) &#124;
