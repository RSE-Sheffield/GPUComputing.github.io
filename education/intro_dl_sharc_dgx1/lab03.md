---
layout: page
title: Lab 03 Deploying and using your trained model
permalink: /education/intro_dl_sharc_dgx1/lab03/
---

# Lab 3: Deploying and using your trained model #

**Remember to be working from the root directory of DLTraining code sample throughout all practicals.**

## Job scripts ##
**A reminder to add the following lines to all the job script that you submit with `qsub`:**

```
#!/bin/bash
#$ -l gpu=1 -P rse-training -q rse-training.q -l rmem=10G -j y

module load apps/caffe/rc5/gcc-4.9.4-cuda-8.0-cudnn-5.1


#Your code below....
```

The `-j y` option is included so that the job output prints everything to one output file e.g. `your_scriptname.sh.o<jobid>`.

## Altering the model file for deployment ##
We'll use the previous MNIST LeNet model that you've just trained.

Copy the `mnist_lenet.prototxt` to `mnist_lenet_deploy.prototxt`.

```
cp mnist_lenet.prototxt mnist_lenet_deploy.prototxt
```

Open the `mnist_lenet_deploy.prototxt` for editing.

For model's deployment, `Data` layers also need to be replaced with one that accepts user input. There's also no need for `Accuracy` and `SoftmaxWithLoss` layer.

Start with removing the `Data` layers name `mnist` for both training and testing phases and replace it with:

```
layer {
    name: "data"
    type: "Input"
    top: "data"
    input_param { shape: { dim: 1 dim: 1 dim: 28 dim: 28 } }
}
```

Remove the `accuracy` and `loss` layer. Replace with a new loss layer that only does Softmax:

```
layer {
  name: "loss"
  type: "Softmax"
  bottom: "ip2"
  top: "loss"
}
```

**Note:** You can check your model file against `code/lab03/mnist_deploy.prototxt`.

## Create a Python interface ##

Now create a new file `mnist_deploy.py` and start editing.

We first import sys, numpy and caffe. Blobs in Caffe are in numpy format:

```
import sys
import numpy as np
import caffe
```

Declare the paths to the deploy model file we've just created and the pre-trained weights from Lab 2:

```
model_path = "mnist_lenet_deploy.prototxt"
weights_path = "mnist_lenet_iter_10000.caffemodel"
```

We'll use CPU mode for now:

```
caffe.set_mode_cpu() # Set as CPU mode
```

We then get Caffe to load our net:

```
#Net loading parameters changed in Python 3
net = caffe.Net(model_path, 1, weights=weights_path)
```

And caffe has a function that loads our image in to a numpy float array:

```
image = caffe.io.load_image(sys.argv[1], False) #Loads the image from the first argument variable
```

We also need to transform the data. In our previous example, in the data layer the input was transformed within the data layer. We will do it using a `Transformer` this time.

```
transformer = caffe.io.Transformer({'data': net.blobs['data'].data.shape})
transformer.set_transpose('data', (2,0,1))  # move image channels to outermost dimension
transformer.set_raw_scale('data', 255)      # rescale from [0, 1] to [0, 255]
transformed_image = transformer.preprocess('data', image)
```

We can now add the transformed image data in to the `data` blob

```
# copy the image data into the memory allocated for the net
net.blobs['data'].data[...] = transformed_image
```

And start training with `net.forward()` which returns an `output` blob.

```
### perform classification
output = net.forward()
output_prob = output['loss'][0]  # the output probability vector for the first image in the batch
```

Below is a trivial example for printing the digits found as a text, in many cases you will have text classifications such as "dogs", "birds", etc. instead:

```
# Text representation of the digit
digits_label = ["Zero","One","Two","Three","Four","Five","Six","Seven","Eight","Nine"]

#Find the index with the highest probablility
highest_index = -1
highest_probability = 0.1 #If nothing's more than 10% sure then don't print out anything

for i in range(0,9):
    if output_prob[i] > highest_probability:
        highest_index = i
        highest_probability = output_prob[i]

#Print our result
if highest_index < 0:
    print("Did not detect a number!")
else:
    print("Digit "+ str(digits_label[highest_index]) + " detected with " + str(highest_probability*100.0)+"%  probability.")
```

Save and create a batch script named `mnist_deploy.sh` with the command:

```
#!/bin/bash
module load apps/caffe/rc5/gcc-4.9.4-cuda-8.0-cudnn-5.1



python mnist_deploy.py data/minist_six.png
```

Run with `qsub`:

```
qsub mnist_deploy.sh
```

## Using the GPU ##

As with training, you can also use the GPU when deploying the model for classification.

Just below the line:

```
net.forward()
```

Add the following lines :

```
caffe.set_mode_gpu()
caffe.set_device(0)
net.forward()
```

Now the training will be run twice, first time with CPU and second with the GPU. Let's also try to time how long it takes by first importing the `time` package:

```
import time
```

And surround the `net.forward()` lines like this:

```
startTime = time.time()
net.forward()
endTime = time.time()
print("Inferencing with CPU took {:.2f}ms".format((endTime-startTime)*1000.0))
```

How much faster was it when using the GPU? Did you get something like this?:

```
Inferencing with CPU took 18.85ms

Inferencing with GPU took 3.25ms
```



## Exercise 3: Batching Inputs ##

It is possible to batch inputs using `reshape` function on a blob. For example:

```
net.blobs['data'].reshape(50,        # batch size
                          3,         # 3-channel (BGR) images
                          227, 227)  # image size is 227x227
```

Multiple loaded images can be added to the data like so:

```
net.blobs['data'].data[0,...] = transformed_image1
net.blobs['data'].data[1,...] = transformed_image2
```

Try applying this to your existing code by batching the following images at the same time and print out all of the predictions:

```
data/mnist_three.png
data/mnist_six.png
data/mnist_nine.png
```

---

&#124; [Home](../) &#124; [Lab02](../lab02) &#124; [Lab04](../lab04) &#124;
