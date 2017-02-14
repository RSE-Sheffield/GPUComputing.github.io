---
layout: page
title: Lab 02 Convolution Neural Network
permalink: /education/intro_dl_sharc_dgx1/lab02/
---

# Lab 02: Convolution Neural Network #

In this lab we will put together a convolution model that can identify handwritten digits by learning from the [MNIST database](http://yann.lecun.com/exdb/mnist/) with a much higher accuracy.

## Running the pre-made model ##

Submit the job file `code/lab02/mnist_lenet_job.sh` using `qsub`:

```
qsub code/lab02/mnist_lenet_job.sh
```


Once the job has finished, check the output for more information in the file `mnist_lenet_job.sh.o<jobid>`, at the end of the file you should get something like below

```
I0127 16:04:25.357823  9366 solver.cpp:317] Iteration 10000, loss = 0.207118
I0127 16:04:25.357884  9366 solver.cpp:337] Iteration 10000, Testing net (#0)
I0127 16:04:25.408476  9366 solver.cpp:404]     Test net output #0: accuracy = 0.9924
I0127 16:04:25.408501  9366 solver.cpp:404]     Test net output #1: loss = 0.279165 (* 1 = 0.279165 loss)
I0127 16:04:25.408510  9366 solver.cpp:322] Optimization Done.
I0127 16:04:25.408516  9366 caffe.cpp:254] Optimization Done.
```

The accuracy of the model is much better at 99.24%.

## Convolution and Pooling layers ##

The `Convolution` layer is defined similarly to the `InnerProduct` layer but instead of `inner_product_param` we have `convolution_param`:

```
layer {
  name: "conv1"
  type: "Convolution"
  bottom: "data"
  top: "conv1"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  convolution_param {
    num_output: 20
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
```

In `convolution_param` the `num_output` variable refers to the number of feature maps to use. `kernel_size` of 5 means a 5x5 convolution and `stride` of 1 means the convolution areas shifts by 1 each time.

For an input image size of 28x28 with kernel size of 5 and stride of 1, each feature map will be of size 24x24 (generating a Nx20x24x24 top blob, with N being number of samples in the batch). Zero-padding can also be added, using `pad: 1` will add a pixel to each side of the input image.

**Note:** Separate width and height dimensions can be specified for 2D convolutions. Use kernel `kernel_w, kernel_w`, stride `stride_w, stride_h` and padding `pad_w, pad_h`.

You will want to make sure that dimensions of the feature map is a whole number. A handy equation for calculating feature map size is:

```
feature_map_width = (width-kernel_size+2*pad)/stride +1
```

A `Pooling` layer is used to perform subsampling on the convolution layer and reduce the network's dimensionality.

```
layer {
  name: "pool1"
  type: "Pooling"
  bottom: "conv1"
  top: "pool1"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
```

In this example a `MAX` pooling is used which selects the highest activation value inside each kernel. The previous convolution layer has feature map size of 24x24 so using `kernel_size: 2` and `stride: 2` we get an Nx20x12x12 output blob.

## Exercise 2: Implementing the LeNet CNN model and Solver ##

Try to implement the LeNet model shown in the following diagram:

{: .center}
![LeNet MNIST diagram](/static/img/intro_dl_sharc_dgx1/mnist_lenet.jpg)

Which looks like the following in netscope:

{: .center}
![Netscope LeNet](/static/img/intro_dl_sharc_dgx1/mnist_lenet_netscope.png)

*Notice that in this model no activation layers are used after the convolution or pooling layers and that Caffe does not implicitly add activation layers. When implementing models such as ConvNet which use activation layers after convolutions, they have to be explicitly added*.

 Use the previous model as a starting point. The `Data`, `SoftmaxWithLoss` and `Accuracy` layers remains the same, there's also no need to keep the `Flatten` layer. Save the model file with name `mnist_lenet.prototxt`.

Check your model file against `code/lab02/mnist_lenet.prototxt` file to see if you've correctly implemented the model.

Once the model's done, copy the previous model's solver file and name it `mnist_lenet_solver.prototxt`. Edit your solver so that it points to your new convolution model. Don't forget to change the `snapshot_prefix` value to reflect the new model name.

Create a batch script to tell caffe to train the model. The result you get should be similar to the pre-made model with the accuracy of around 99.2%


## Resume training ##
Snapshots are taken every 5000 iterations according to the solver file (`snapshot: 5000`), this generates a `.caffemodel` file that contains model weights and a `.solverstate` file that contains all information to resume training from that point. The files can be used to continue training the model should something go wrong while training (power cuts etc.).

Use the `-snapshot` flag to include snapshots in your training e.g.:

```
caffe test -solver mnist_lenet_solver.prototxt -snapshot mnist_lenet_iter_5000.solverstate
```


## Plotting your training and testing progress ##

A modified version of Caffe's tool for parsing output logs are included in the `tools` directory.

Use the `tools/parse_log.sh` on the the generated log file to create a `.train` and `.test` files with the training and test statistics.

```
tools/parse_log.py mnist_lenet_train.sh.o<jobid>
```

Use the `tools/plot_log.py` on one of the generated files to see what fields are available to plot against:

```
$ tools/plot_log.py mnist_lenet_train.sh.o<jobid>.train
Available headers:
 Iters
 Seconds
 TrainingLoss
 LearningRate
```

To generate a plot, simply add the correct header to use for X and Y axis:

```
$ tools/plot_log.py mnist_lenet_train.sh.o<jobid>.train Iters TrainingLoss mnist_lenet_plot.png
```

The above code produces the graph shown below. An optional `.png` image was also saved.

{: .center}
![Mnist lenet Iteration vs TrainingLoss plot](/static/img/intro_dl_sharc_dgx1/mnist_lenet_iters_vs_trainingloss_plot.png)

## Extra: Full Convolution Networks and Object Detection ##

* Approaches to Object Detection using DIGITS at [https://nvidia.qwiklab.com/](https://nvidia.qwiklab.com)
  * Register at the site and pass on your username/e-mail so you can be forwarded with credits for the course
* Caffe's tutorial on Object detection using R-CNN [https://github.com/BVLC/caffe/blob/master/examples/detection.ipynb](https://github.com/BVLC/caffe/blob/master/examples/detection.ipynb)
  * Code for downloading data can be found in the Caffe repository:
    ```
    git clone https://github.com/BVLC/caffe.git
    ```

---

&#124; [Lab01](../lab01) &#124; [Lab03](../lab03) &#124;
