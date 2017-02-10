---
layout: page
title: Lab 04 Using and visualising pre-trained models
permalink: /education/intro_dl_sharc_dgx1/lab04/
---

# Lab 04: Using and visualising pre-trained models #

The ImageNet competition provides a dataset of tagged images collected from the internet. The whole set is around 55GB and can take days or weeks to train.

In this lab we will look at using a pre-trained models provided by Caffe and its community, in particular the Caffenet model that was based on Alexnet architecture that won the 2012 ImageNet competition. In addition we will explore how the model parameters can be visualised using the Python interface.

## Caffe model zoo ##

Caffe provides a directory of official and community trained models. They can be found at the [model zoo](http://caffe.berkeleyvision.org/model_zoo.html) page.


## Getting the pre-trained Caffenet model ##

All necessary files are included in `code/lab04/caffenet` folder apart from the pre-traind binary file which is too large to store in the repository (over 240MB). To get the file use:

```
wget "https://www.dropbox.com/s/srmy7h9bqy25vaq/bvlc_reference_caffenet.caffemodel?dl=0"  -O data/lab04/caffenet/bvlc_reference_caffenet.caffemodel
```

The model is more complex than our previous LeNet model using five stages of convolution and Local Response Normalization `LRN` layers which introduces lateral inhibition for bounding response of unbounded activation functions such as ReLU.

![Alexnet diagram](/static/img/intro_dl_sharc_dgx1/alexnet_small.png)


## Exercise 4: Deploying the Caffenet model for classification ##

Using the previous python deployment code for our LeNet model. Modify it so that it can classify the `data/cat.jpg` image then print out the text labels with associated probabilities. See guidance below on the necessary steps:

* The model file is located at `code/lab04/caffenet/deploy.prototxt`
* The weights file is located at `code/lab04/caffenet/bvlc_reference_caffenet.caffemodel`
* The model takes an image input of `227 width x 227 height x 3 channels (RGB)`
  * The channel must be swapped from RGB to BGR
      ```
      transformer.set_channel_swap('data', (2,1,0))
      ```
* The mean value file is located at `code/lab04/caffenet/ilsvrc_2012_mean.npy`
  * The mean file can be loaded like so

    ```
    mu = np.load('code/lab04/caffenet/ilsvrc_2012_mean.npy')
    mu = mu.mean(1).mean(1)  # average over pixels to obtain the mean (BGR) pixel values
    ```

  * The transformer can take in the mean value before the image is transformed

    ```
    transformer.set_mean('data', mu)
    ```

* Text labels are located at `code/lab04/caffenet/synset_words.txt`, numpy can be used to load the set:
  ```
  labels = np.loadtxt(`code/lab04/caffenet/synset_words.txt`, str, delimiter='\t')
  ```

When done, you can check your code against `code/lab04/caffenet_deploy.py`.

## Examining Blobs ##

The `net.blobs` object is an `OrderedDict` containing Blob objects with its name as the key. Add the following code to your Python script to print out all blob dimensions:

```
# for each layer, show the output shape
for layer_name, blob in net.blobs.items():
    print(layer_name + '\t' + str(blob.data.shape))

```

Where you will get:

```
data	(50, 3, 227, 227)
conv1	(50, 96, 55, 55)
pool1	(50, 96, 27, 27)
norm1	(50, 96, 27, 27)
conv2	(50, 256, 27, 27)
pool2	(50, 256, 13, 13)
norm2	(50, 256, 13, 13)
conv3	(50, 384, 13, 13)
conv4	(50, 384, 13, 13)
conv5	(50, 256, 13, 13)
pool5	(50, 256, 6, 6)
fc6	(50, 4096)
fc7	(50, 4096)
fc8	(50, 1000)
prob	(50, 1000)
```

## Examining model parameters ##

Similarly the `net.params` contain the parameters of the model. To print our their dimension use the following code:

```
for layer_name, param in net.params.items():
    print(layer_name + '\t' + str(param[0].data.shape), str(param[1].data.shape))
```

Where you will get:

```
conv1	(96, 3, 11, 11) (96,)
conv2	(256, 48, 5, 5) (256,)
conv3	(384, 256, 3, 3) (384,)
conv4	(384, 192, 3, 3) (384,)
conv5	(256, 192, 3, 3) (256,)
fc6	(4096, 9216) (4096,)
fc7	(4096, 4096) (4096,)
fc8	(1000, 4096) (1000,)
```

## Visulisation of model parameters ##

Let's try to visualise these parameters. Add the following function in to your script:

```
def vis_square(data):
    """Take an array of shape (n, height, width) or (n, height, width, 3)
       and visualize each (height, width) thing in a grid of size approx. sqrt(n) by sqrt(n)"""

    # normalize data for display
    data = (data - data.min()) / (data.max() - data.min())

    # force the number of filters to be square
    n = int(np.ceil(np.sqrt(data.shape[0])))
    padding = (((0, n ** 2 - data.shape[0]),
               (0, 1), (0, 1))                 # add some space between filters
               + ((0, 0),) * (data.ndim - 3))  # don't pad the last dimension (if there is one)
    data = np.pad(data, padding, mode='constant', constant_values=1)  # pad with ones (white)

    # tile the filters into an image
    data = data.reshape((n, n) + data.shape[1:]).transpose((0, 2, 1, 3) + tuple(range(4, data.ndim + 1)))
    data = data.reshape((n * data.shape[1], n * data.shape[3]) + data.shape[4:])

    plt.imshow(data); plt.axis('off')
```

For `conv1` layer's parameter:

```
# the parameters are a list of [weights, biases]
filters = net.params['conv1'][0].data
vis_square(filters.transpose(0, 2, 3, 1))
```

![Caffenet conv1 params](/static/img/intro_dl_sharc_dgx1/caffenet_conv1_param.png)

The first layer output, conv1 (rectified responses of the filters above, first 36 only):

```
feat = net.blobs['conv1'].data[0, :36]
vis_square(feat)
```

![Caffenet conv1 blob](/static/img/intro_dl_sharc_dgx1/caffenet_conv1_blob.png)


The fifth layer after pooling, pool5:

```
feat = net.blobs['pool5'].data[0]
vis_square(feat)
```

![Caffenet pool5 blob](/static/img/intro_dl_sharc_dgx1/caffenet_pool5_blob.png)

The first fully connected layer, fc6 (rectified). We show the output values and the histogram of the positive values:

```
feat = net.blobs['fc6'].data[0]
plt.subplot(2, 1, 1)
plt.plot(feat.flat)
plt.subplot(2, 1, 2)
_ = plt.hist(feat.flat[feat.flat > 0], bins=100)
```

![Caffenet fc6](/static/img/intro_dl_sharc_dgx1/caffenet_fc6_histogram.png)

The final probability output, prob:
```
feat = net.blobs['prob'].data[0]
plt.figure(figsize=(15, 3))
plt.plot(feat.flat)
```

![Caffenet output prob](/static/img/intro_dl_sharc_dgx1/caffenet_prob_distribution.png)

Note the cluster of strong predictions; the labels are sorted semantically. The top peaks correspond to the top predicted labels, as shown above.


## Extra: Fine-tuning and rehaping pre-train models ##

Caffe has provided tutorials for using an network trained on ImageNet data and repurposing it for classifying flickr style. To use the tutorial you will want to clone Caffe from github

```
git clone https://github.com/BVLC/caffe.git
```

And follow the tutorial book in iPython format here: [https://github.com/BVLC/caffe/blob/master/examples/02-fine-tuning.ipynb](https://github.com/BVLC/caffe/blob/master/examples/02-fine-tuning.ipynb)

Another tutorial is also provided for re-shaping an existing network:
[https://github.com/BVLC/caffe/blob/master/examples/net_surgery.ipynb](https://github.com/BVLC/caffe/blob/master/examples/net_surgery.ipynb)

---

&#124; [Lab03](../lab03) &#124; [Lab05](../lab05) &#124;
