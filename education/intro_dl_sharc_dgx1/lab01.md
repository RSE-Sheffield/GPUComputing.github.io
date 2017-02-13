---
layout: page
title: Lab 01 A simple MNIST model
permalink: /education/intro_dl_sharc_dgx1/lab01/
---

# Lab 01: A simple MNIST model #

In this lab we will put together a basic 3-layer model that can identify handwritten digits by learning from the [MNIST database](http://yann.lecun.com/exdb/mnist/).


## Running the pre-made model ##

Submit the job file `code/lab01/mnist_job.sh` using `qsub`:

```
qsub code/lab01/mnist_job.sh
```

Once the job has finished, check the output for more information in the file `mnist_job.sh.o<jobid>` at the end of the file you should get something like below

```
I0127 16:04:25.357823  9366 solver.cpp:317] Iteration 10000, loss = 0.207118
I0127 16:04:25.357884  9366 solver.cpp:337] Iteration 10000, Testing net (#0)
I0127 16:04:25.408476  9366 solver.cpp:404]     Test net output #0: accuracy = 0.9226
I0127 16:04:25.408501  9366 solver.cpp:404]     Test net output #1: loss = 0.279165 (* 1 = 0.279165 loss)
I0127 16:04:25.408510  9366 solver.cpp:322] Optimization Done.
I0127 16:04:25.408516  9366 caffe.cpp:254] Optimization Done.
```

You've just trained a basic neural network model on Caffe! The accuracy should be around 92%.

Let's have a look at how caffe is used from the command line, type in:

```
cat code/lab01/mnist_job.sh
```

In the script you will see the line with `caffe train`:

```
caffe train -solver=code/lab01/mnist_simple_solver.prototxt
```

Caffe offers a command line interface for training your models. The above command indicates that you will be training the model solver file located at `code/lab01/mnist_simple_solver.prototxt`.


## Implementing the model ##

Two text files are needed to get a model running in Caffe. A model file which defines the architecture of the network and a solver file that lets you choose the approach to training and optimisation.

A model file consists of sequences of layers each with specific functionality such as `Data` layer that allows import of external data from raw images or databases such as LMDB or the Loss layer that calculates the error/loss function. Blobs are matrices for storing data and are used as input (`bottom`) and generated as outputs (`top`) of layers. Layers can have multiple blobs as inputs or outputs depending on the type. The model file is written in the format of Google's protocol buffer (protobuf).

To get a feel for the model file we will implement a very simple model that trains on the mnist data with only one dense hidden layer.

![mnist simple](/static/img/intro_dl_sharc_dgx1/mnist_simple.png)

We'll use the Netscope to visualise our network as we make it. Open the following link in a new tab/window to get started: [http://ethereon.github.io/netscope/#/editor](http://ethereon.github.io/netscope/#/editor).

First name the model:

```
name: "MNIST Simple"
```

Then add a data layer:

```
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "data/mnist_train_lmdb"
    batch_size: 64
    backend: LMDB
  }
}
```

This creates a `Data` layer named "mnist" that reads from an LMDB database `source: "data/mnist_train_lmdb"` **note that the location of files referenced in a script is relative to where you call Caffe and not the location of the script itself**. The layer has two outputs, `top: "data"` blob is the image data and `top: "label"` is the correctly categorised label in a one-hot format (an array with a 1 value for the correct category and 0 for the others). For training we'll use a batch size of 64 `batch_size: 64`.

Each input pixel has range of 0-255 so we need to scale it to the range 0-1 by multiplying with `scale: 0.00390625` (1/256) in the `transform_param`.

The input data is arranged as a 2D grid which we'll use for our next lab. For now we'll flatten it to a 1-dimensional array using a `Flatten` layer with `data` blob as input and `flatdata` blob as output:

```
layer {
	name: "flatdata"
	type: "Flatten"
	bottom: "data"
	top: "flatdata"
}
```

Now add an `InnerProduct` layer, this is essentially a 'dense' layer where all nodes are connected to every other node below:

```
layer {
	name: "ip"
	type: "InnerProduct"
	bottom: "flatdata"
	top: "ip"
	param {
		lr_mult: 1
	}
	param {
		lr_mult: 2
	}
	inner_product_param {
		num_output: 10
		weight_filler {
			type: "xavier"
		}
		bias_filler {
			type: "constant"
		}
	}
}
```

The `InnerProduct` layer `ip` takes the `flatdata` blob as input and generates `ip` blob as output. The `num_ouput` is 10 in this case as we have 10 classifiers (digits 0-9).

The fillers (`weight_filler`, `bias_filler`) allow us to randomly initialize the value of the weights and bias. For the `weight_filler`, we will use the [xavier](http://andyljones.tumblr.com/post/110998971763/an-explanation-of-xavier-initialization) algorithm that automatically determines the scale of initialisation based on the number of input and output neurons. For the `bias_filler`, we will simply initialise it as constant, with the default filling value 0.

`lr_mult`s are the learning rate adjustments for the layer’s learnable parameters. In this case, we will set the weight learning rate to be the same as the learning rate given by the solver during runtime, and the bias learning rate to be twice as large as that - this usually leads to better convergence rates.

Now we just need to add a suitable activation function for classification (Softmax) and and loss calculation to our model. Caffe provides a layer that does both for us with `SoftmaxWithLoss` layer.

```
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "ip"
  bottom: "label"
  top: "loss"
}
```

The `loss` layer takes the 2 input blobs `ip` and `label` and generates a `loss` blob.

That's all we need to create a model for training. Press `Shift+Enter` to view the network. You should get something like this:

![simple mnist](/static/img/intro_dl_sharc_dgx1/mnist_simple_netscope.jpg)

Rules can be added to layers to specify when they're included in to a network. The `phase` rule indicates whether the layer will be included in either the Training or the Testing phase

```
	layer {
	  // ...layer definition...
	  include: { phase: TRAIN }
	}
```

After a certain number of training epochs we will have the model use the test data to verify the model's accuracy. Replace the existing data layer with:

```
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "data/mnist_train_lmdb"
    batch_size: 64
    backend: LMDB
  }
}
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "data/mnist_test_lmdb"
    batch_size: 100
    backend: LMDB
  }
}
```

We now have 2 data layers that reads from the `mnist_train_lmdb` in the training phase and from `minist_test_lmdb` in the testing phase.

Adding an `Accuracy` utility layer in the testing phase makes Caffe calculate and output accuracy values.

```
layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "ip"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
```

Your final model file should look something like this:

```
name: "MNIST Simple"
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "data/mnist_train_lmdb"
    batch_size: 64
    backend: LMDB
  }
}
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    scale: 0.00390625
  }
  data_param {
    source: "data/mnist_test_lmdb"
    batch_size: 100
    backend: LMDB
  }
}
layer {
	name: "flatdata"
	type: "Flatten"
	bottom: "data"
	top: "flatdata"
}

layer {
  name: "ip"
  type: "InnerProduct"
  bottom: "flatdata"
  top: "ip"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}

layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "ip"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "ip"
  bottom: "label"
  top: "loss"
}

```

Create a file `mnist_simple.prototxt` and paste the content of the model in to it. We can now start to implement the solver.

To see what other layers Caffe supports, see the [layer catalogue](http://caffe.berkeleyvision.org/tutorial/layers.html).


## Implementing the solver ##

The solver file lets us define how the training and testing is performed.

Create a `mnist_simple_solver.prototxt` file and copy in the code below:

```
	# The train/test net protocol buffer definition
	net: "mnist_simple.prototxt"

	# Declare solver type, SGD is Stochastic Gradient Descent
	type: SGD

	# test_iter specifies how many forward passes the test should carry out.
	# In the case of MNIST, we have test batch size 100 and 100 test iterations,
	# covering the full 10,000 testing images.
	test_iter: 100

	# Carry out testing every 500 training iterations.
	test_interval: 500

	# The base learning rate, momentum and the weight decay of the network.
	base_lr: 0.01
	momentum: 0.9
	weight_decay: 0.0005

	# The learning rate policy
	lr_policy: "inv"
	gamma: 0.0001
	power: 0.75

	# Display every 100 iterations
	display: 100

	# The maximum number of iterations
	max_iter: 10000

	# snapshot intermediate results
	snapshot: 5000
	snapshot_prefix: "intro_dl_snapshot_"

	# solver mode: CPU or GPU
	solver_mode: GPU
```

See the comments in the code for more details.

Fore more information on Caffe's available solvers http://caffe.berkeleyvision.org/tutorial/solver.html.

## Training ##

To train the model create a new script `mnist_simple_train.sh` with the contents

```
#!/bin/bash
#$ -l gpu=1 -P rse-training -q rse-training.q -l rmem=10G

module load libs/caffe/rc3/gcc-4.9.4-cuda-8.0-cudnn-5.1-conda-3.4-TESTING
source activate caffe

caffe train -solver=mnist_simple_solver.prototxt
```

Submit the job using `qsub`. In the output file you should get similar results to the pre-built model with final accuracy of around 92%.

Training process
	Feeding forward
	Backprop
	Stochastic Batching

Why are the results not the same every time?
	Because of stohasticity in variable initialization and also when doing gradient descent

When do we stop training?
	Guesswork
	Check gradient of loss

## Activation Layers ##

Our current model has `Softmax` (sigmoid) rolled in to the loss layer to introduce non-linearity but as you start to add additional layers, activation functions has to be added manually.

The `ReLU` (rectified linear unit) is a popular activation function filters out values below 0. It reduces the chance of vanishing gradients and has been found to converge faster than sigmoid type functions.

![ReLU](/static/img/intro_dl_sharc_dgx1/relu_function.png)

![ReLU plot](/static/img/intro_dl_sharc_dgx1/relu_plot.jpg)

The `ReLU` layer can be added to the model with

```
layer {
	name: "relu"
	type: "ReLU"
	bottom: "a_blob"
	top: "a_blob"
}
```
*Note that you can use the same blob name for both top and bottom to do the computation in-place and save memory by not creating a new blob.*

See the [layer catalogue](http://caffe.berkeleyvision.org/tutorial/layers.html) for more activation layers.

## Exercise 1.1: Different solvers ##

Try using different solvers such as `Adam` or `RMSProp`. Does the model converge faster? (See the [solver page](http://caffe.berkeleyvision.org/tutorial/solver.html) for specific parameters required for each solver type.)

## Exercise 1.2: Additional Layers ##

Try adding additional `InnterProduct` layer(s) to the current network, does the accuracy improve? (Don't forget to add activation functions.)


---

&#124; [Getting Started](../getting_started) &#124; [Lab02](../lab02) &#124;
