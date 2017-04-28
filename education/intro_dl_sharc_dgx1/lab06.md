---
layout: page
title: Lab 06 Recurrent Neural Networks
permalink: /education/intro_dl_sharc_dgx1/lab06/
---

# Practical 6: Recurrent Neural Networks #

**Remember to be working from the root directory of DLTraining code sample throughout all practicals.**

In this lab we will create a simple text generation model that allows us to see the capabilities of the Long Short Term Memory (LSTM) units. It will include creating a HDF5 data set from raw sample text 'Alice in Wonderland' as well as creating a simple script for generating a random text sequence from the output of our trained model. The lab introduces 4 new layers, the `HDF5Data`, `LSTM`, `Embed` and `Dropout` layers.

## Job scripts ##
**A reminder to add the following lines to all the job script that you submit with `qsub`:**

```
#!/bin/bash
#$ -l gpu=1 -P rse-training -q rse-training.q -l rmem=10G -j y

module load apps/caffe/rc5/gcc-4.9.4-cuda-8.0-cudnn-5.1


#Your code below....
```

The `-j y` option is included so that the job output prints everything to one output file e.g. `your_scriptname.sh.o<jobid>`.

## LSTM Layer ##

The `LSTM` is a layer that takes in a sequence of input over time steps and can be used to generate a sequence of output over time.


```
layer {
  name: "lstm1"
  type: "LSTM"
  bottom: "input_sequence"
  bottom: "continuity_sequence"
  top: "lstm1"
  recurrent_param {
    num_output: 256

    weight_filler {
    type: "xavier"
     }
  	bias_filler {
  		type: "constant"
  	}
  }
}
```

{: .center}
![LSTM](/static/img/intro_dl_sharc_dgx1/lstm.png)

`LSTM` takes in a minimum of 2 blobs as input, the first blob is an input sequence which is arranged as `TxNx...` matrix where `T` is the timestep sequence, `N` is the number of independen streams. The second blob takes in a "sequence continuation indicator" where `0` indicates the start of a new sequence and `1` indicates a continuation of the sequence. It is structured as `TxN` matrix. `num_output` indicates how memory units it has.

The layer also has an  *optional* input with dimensions `Nx...` which is a static input. It can be used to input features which do not change over time.

## The Embedding Layer ##

The `Embed` layer transforms a one-hot encoded input of size `input_dim` and maps it as a vector in continous space of size `num_output`.

{: .center}
![Embedding](/static/img/intro_dl_sharc_dgx1/embedding.png)

```
layer {
  name: "embedding"
  type: "Embed"
  bottom: "input"
  top: "embedded_input"
  param {
    lr_mult: 1
  }
  embed_param {
    bias_term: false
    input_dim: 8801  # = vocab_size
    num_output: 1000
    weight_filler {
		  type: "xavier"
	  }
  }
}
```

This layer will be used to embed our character input generated from the sample text.

If you would like to know more about word embedding the blog pages [here](https://blog.acolyer.org/2016/04/21/the-amazing-power-of-word-vectors) and  [here](http://sebastianruder.com/word-embeddings-1) are good starting points.

## The Droput Layer ##

The `Dropout` layer disables a random number of neurons as indicated by the `dropout_ratio` during training to prevent overfitting. It applies the dropout only to the input `bottom` layer.

```
layer {
  name: "lstm-drop"
  type: "Dropout"
  bottom: "lstm1"
  top: "lstm1"
  dropout_param { dropout_ratio: 0.35 }
}
```

*The `Dropout` layer works only during training and has no effect when testing or performing inferencing.

## The HDF5Data Layer ##

Unlike the `Database` layer, the `HDF5Data` layer can generate more than two top blobs for use as inputs in to our network. The name of the top blobs **must** correspond with the key value specified in our database.

The `source` variable specifies the location of a test file that contains a line-separated list of paths to the `.hdf5` file. This allows the loading of large databases that has to be chunked in to smaller parts.

```
layer{
	name: "data"
	type: "HDF5Data"
	top: "input_sequence"
	top: "cont_sequence"
	top: "target_sequence"
	hdf5_data_param {
		source: "wonderland_hdf5_list.txt"
		batch_size: 50
	}
}
```

## Preparing the data ##

A copy of 'Alice in Wonderland' is located at `data/wonderland.txt`. We will be using this to generate a HDF5 data set for our model. The text generation model accepts a sequence of characters, a sequence of continuity delta and a target output sequence of characters:

![Alice data](/static/img/intro_dl_sharc_dgx1/alice_continuity.png)

Start by installing the `h5py` package:

```
module load apps/caffe/rc5/gcc-4.9.4-cuda-8.0-cudnn-5.1



pip install h5py
```

Create a python file `create_text_gen_dataset.py` add the start importing packages that we'll need:

```
import h5py
import numpy as np
import json
```

Declare out the input and output file paths:

```
filename = "data/wonderland.txt"
dict_output_file = "wonderland_dict.json"
hdf_output_file = "wonderland.hdf5"
hdf_list_file = "wonderland_hdf5_list.txt"
```

Open the file `data/wonderland.txt`, read the text and convert it to lowercase:

```
raw_text = open(filename).read()
raw_text = raw_text.lower()
print("Raw text length: ", len(raw_text))
```

We'll limit the dataset to `50,000` characters `total_length` and number of independent streams `num_streams` we'll train at the same time (`N` dimension of LSTM) to be  `250`. We then get the `stream_length` by dividing them together:

```
total_length = int(50000)
num_streams = int(250)
stream_length = int(total_length/num_streams)
```

Create a `clipped_text` that only has the text in the dataset plus one (for prediction):

```
clipped_text = raw_text[0:total_length+1]
```

A mapping of all unique charaters to integers are created. `set()` reduces the string to only unique characters, convert to a list with `list()` and `sorted()` arranges the character in order. A dictionary is then created to map characters to their integer(index) value:

```
chars = sorted(list(set(clipped_text)))
char_to_int = = dict((c, i) for i, c in enumerate(chars))
```

At this point we also want to save this character mapping for use when inferencing. The dictionary is serialised to a JSON format:

```
with open(dict_output_file, "w") as f:
    f.write(json.dumps(char_to_int))
    f.close()
```

We'll want to know how many characters have in the dictionary:

```
n_chars = len(clipped_text)
n_vocab = len(chars)
print("Total Characters: ", n_chars)
print("Total Vocab: ", n_vocab)
print("Text length: ", total_length , " stream length: ", stream_length, " numstreams: ", num_streams)
```

Create 2D matrices of input `input_data`, continuity `cont_data` and target data `target_data`. For convenience we are creating a matrix of `NxT` first:

```
input_data = []
cont_data = []
target_data = []

for i in range(num_streams):
    input_stream = []
    cont_stream = []
    target_stream = []
    for j in range(stream_length):
        text_index = i*stream_length + j
        char_in = clipped_text[text_index]
        char_target = clipped_text[text_index+ 1]
        input_stream.append(char_to_int[char_in])
        cont_stream.append(1)
        target_stream.append(char_to_int[char_target])
    input_data.append(input_stream)
    cont_data.append(cont_stream)
    target_data.append(target_stream)
```

We're not worried about continuity data for our model so will always use `1` in `cont_stream` to indicate that everything belongs to the same sequence.

Notice that we're simply passing in a single integer value in to the `input_stream` and `target_stream`. Caffe knows to expand this value in to a one-hot representation for us as we'll have to specify the `input_dim` in the `Embed` layer and `num_output` in the last `InnerProduct` layer.

Then we can convert them to numpy arrays and transpose their axis in to the correct `TxN` dimension:

```
input_np = np.array(input_data, dtype='float32')
input_np = np.transpose(input_np, (1, 0))

cont_np = np.array(cont_data, dtype='uint8')
cont_np = np.transpose(cont_np, (1,0))

target_np = np.array(target_data, dtype='float32')
target_np = np.transpose(target_np, (1, 0))

print("Input data shape: ", input_np.shape)
print("Cont data shape: ", cont_np.shape)
print("Target data shape: ", target_np.shape)
```

Create a hdf5 file with the following:

```
with h5py.File(hdf_output_file, "w") as f:
    #Create dataset
    f.create_dataset("input_sequence", data=input_np)
    f.create_dataset("cont_sequence", data=cont_np)
    f.create_dataset("target_sequence", data=target_np)
    f.close()
```

The name given to the dataset's index correspeonds to the `top` of `HDF5Data` layer. So in the model file we will have the `top` layers of `input_sequence`, `cont_sequence` and `target_sequence`.

We also need to create a `.txt` file with the path to our `.hdf5` file:
```
with open(hdf_list_file, "w") as f:
    f.write(hdf_output_file)
    f.close()
```

Multiple files can be included in the text file, they just need to be line separated e.g.:

```
file1.hdf5
file2.hdf5
```

You can check your code against the file `code/lab06/create_text_gen_dataset.py`.

Now run the file to generate the data:

```
$ python create_text_gen_dataset.py
Raw text length:  144436
Total Characters:  50001
Total Vocab:  47
Text length:  50000  stream length:  200  numstreams:  250
Input data shape:  (200, 250)
Cont data shape:  (200, 250)
Target data shape:  (200, 250)
```

*Note that we have 47 unique characters in our dataset, we need to keep this in mind as we design our `Embed` and the last `InnerProduct` layer.*

For completeness, here's how we would open and use a hdf5 file in python:

```
with h5py.File("hdf_file_path.hdf5", "r") as f:
    print(f["input_sequence"].shape)
    print(f["cont_sequence"].shape)
    print(f["target_sequence"].shape)
```

## Exercise 6.1 Implementing the model ##
The diagram below shows the network we'll be implementing:

{: .center}
![LSTM](/static/img/intro_dl_sharc_dgx1/text_gen_lstm.png)

The layer sequence is:
* `HDF5Data`
* `Embed`
* `LSTM`
* `Dropout`
* `InnerProduct`
* `SoftmaxWithLoss`

Create a `text_gen.prototxt` file and try to implement the model using information about the layers above. Here are some guidelines for the implementation:

* There's no testing phase for the model as we are not interested in its accuracy of prediction.
* `HDF5Data` layer
  * Has 3 top blobs
  * Use a batch size of `50`
* `Embed` layer
  * `num_output` can be any arbitrary number lower than the input vocabulary size.
* `LSTM` layer
  * Use `num_output: 256` for now.
* `Dropout` layer
  * Start with `dropout_ratio` of 0.35 to add a good amount of randomisation.
* `InnerProduct` layer
  * In the `inner_product_param` set `axis: 2`. Unlike our previous model, an additional time `T` dimension is added `TxNxC` and an `axis: 2` means we keep the entire output matrix rather than the default `axis: 1` where we would only keep `TxN` matrix.
* `SoftmaxWithLoss`
  * Similarly in the `softmax_param` set `axis:2` to tell the layer to evaluate classification along the last axis.

You can check your model with the file `code/lab06/text_gen.prototxt`.

## Implementing the Solver ##

Create a `text_gen_solver.prototxt` file and use the following content:

```
# The train/test net protocol buffer definition
net: "text_gen.prototxt"

# Declare solver type, SGD is Stochastic Gradient Descent
type: "Adam"


# The base learning rate, momentum and the weight decay of the network.
base_lr: 0.001
momentum: 0.2

weight_decay: 0.0001

# The learning rate policy
lr_policy: "inv"
gamma: 0.00001
power: 0.75

# Display every 100 iterations
display: 100

# The maximum number of iterations
max_iter: 6000

# snapshot intermediate results
snapshot: 2000
snapshot_prefix: "text_gen"

# solver mode: CPU or GPU
solver_mode: GPU

#Apply gradient clipping threshold to all layers
clip_gradients: 1.0
```

An additional variable `clip_gradients` was added, it has been shown to help deal with exploding gradients and vanishing gradients problem \([Pascam et al. 2012 ](https://arxiv.org/abs/1211.5063)\) which occurs more frequently in recurrent networks due to the use of sigmoid function in the recurrent units.

Create a job script for training the model and submit with `qsub`. The loss values will depend on the model variable you've set but for the example model it should be around `0.4`.

## Exercise 6.2 Create a deployable model ##

Create a deployable model file called `text_gen_deploy.prototxt` by replacing `HDF5Data` layer with 2 `Input` layers. Also replace `SoftmaxWithLoss` layer with just the `Softmax` layer.

Check with the file `code/lab06/text_gen_deploy.prototxt` to ensure your model is correct.

## Creating a text generation script ##

With the deployment version of the model completed, we're ready to create a script that generates some text for us.

Create `generate_text.py` file and start importing the necessary modules:

```
import numpy as np
import caffe
import json
import random
```

Add paths to our deployment model and weights and load the model:

```
model_path = "code/lab06/text_gen_deploy.prototxt"
weights_path = "text_gen_iter_6000.caffemodel"
net = caffe.Net(model_path, 1, weights=weights_path)
```

Set GPU mode:

```
caffe.set_mode_gpu()
caffe.set_device(0)
```

Get out previously saved charcter dictionary and make a reverse lookup dictionary:

```
#Gets the json char index map
dict_input_file = "wonderland_dict.json"
char_to_int = json.loads(open(dict_input_file).read())

#And make a reverse char lookup map
int_to_char = dict((i, c) for c, i in char_to_int.items())

num_vocab = len(int_to_char)
print("Total Vocab: ", num_vocab)

```

We'll use the same sequence length as the batching in the model:

```
#Predict sequence
seq_length = 50
no_predict = 1000 #Number of characters to generate
```

Choose a random excerpt from the data:

```
test_file = "data/looking_glass.txt"
raw_text = open(test_file,"r").read()
raw_text_length = len(raw_text)
seed_start = random.randint(0,raw_text_length - seq_length)
seed_text = raw_text[seed_start:seed_start+seq_length].lower()
```

Replace any charcters not used in our training data with a blank space:

```
for i, c in enumerate(seed_text):
	if c not in char_to_int:
		seed_text[i] = " "
```

Create data structures for feeding in to our model:

```
#Gets the input blobs
input_blob = net.blobs['input_sequence']
cont_blob = net.blobs['cont_sequence']

#Create ndarray for filling
input_np = np.zeros( (seq_length, 1), dtype="float32")
cont_np = np.zeros( (seq_length,1) , dtype="float32")
cont_np.fill(1) #Continuity is always 1

#Stores character sequence as we start generation
input_queue= []
```

Fill `input_queue` with seed text:

```
for c in seed_text:
	input_queue.append(c)

print("Seeding with text: \n", "".join(input_queue))
```

Finally we create a loop to generate the text. Conver `input_queue` to an integer representaiton and fed it in to `input_np`. The two data blobs `input_blob` and `cont_blob` are then filled. After running the model `net.forward()` we get an ouptput sequence matrix of size `50x1x47`. The model makes an entire sequence of prediction but we only add the last generated character to our `input_queue` and continue with our text generation:

```
result = ""

#Generate some text
for i in range(no_predict):

	#Fill numpy arrays
	for j in range(seq_length):
		input_np[j,0] = char_to_int[input_queue[j]]

	#Fill the data blob
	input_blob.data[...] = input_np
	cont_blob.data[...] = cont_np
	output = net.forward()
	output_prob = output['probs']

	#Gets all the predicted characters and its confidence
	out_seq = []
	out_conf_seq = []
	for p in output_prob:
		#Gets the index with maximum probability
		out_max_index = p[0].argmax()
		#Get actual charcter
		predicted_char = int_to_char[out_max_index]
		#Get confidence of the prediction
		confidence = p[0,out_max_index]

		#Adds to the array
		out_seq.append(predicted_char)
		out_conf_seq.append(confidence)


	next_char = out_seq[-1]
	next_confidence = out_conf_seq[-1]


	#Print the result of this prediction
	print("Prediction no.: ", str(i))
	print("Input sequence: \n", input_queue)
	print("Output sequence: \n", out_seq)
	print("Output confidence: \n", out_conf_seq)
	print("Next char prediction: ", repr(next_char), " confidence: ", str(next_confidence))

	#Add to the input list and pop the first character
	input_queue.append(next_char)
	popped_char = input_queue.pop(0)

	#Add text to the final string
	result += popped_char

#Fill the rest text with the remaining prediction
for c in input_queue:
	result += c

print("Final output:")
print(result)
```

Check with the file `code/lab06/generate_text.py` to ensure your script is correct.

Create and run a job script with `qsub` when done.

We select random part of the text for seeding so your results will be variable. With the following seed for example:
```
nished: and the trial doesn’t
even begin till next
```

You should get something almost legible:

```
nished: and the trial doesn’t
even begin till next crass all mast cemantyou say, at to liked to conly way
.
alowe toute, ‘curnder ferle sooned lattle white rabed of on the hald was very angar a reall
of atile, and down at of then wher sme the reagrit lessall, ‘ah, i should be litele that it marked and sorewh, ron ture call to the way that sales, and ereh surey was goting on the eagher,
and a doicam as the caterpillar cerlealy to fin her white say it was cotsins, and be sound seem to be sureristhim irosthatalas! cold it against and took the hall was mowend on forth the
harss--and seet, as no cats of thist, but sperad in the window, and of little they, as she swould feel her eats rame-be a get wordo way in a corfustone it as it was look and our her.

‘why, mary ann thes all speend, and had be nowers in a moner--howe near the comfle, on the doon ould mark herself herself to little they was grizt near of took that all ther, she
was to dry be
if she could no reasing to her
very good--bores, i fon’t fintth the rook, and ‘whey seemid out of
```

## Exercise 6.3 Varying dropouts and adding additional LSTM layers ##
* Try changing the dropout values when training
* Try adding additional `LSTM` layers to your model.
  * You can check with `code/lab/06/text_gen_multilayer.prototxt` and `code/lab/06/text_gen_multilayer_solver.prototxt`
  * Don't forget to change the model paths in your `generate_text.py`

## Extra ##
* If you'd prefer to use LMDB instead, Gustav Larsson has provided a very concise tutorial on using them in Python on his [blog page](http://deepdish.io/2015/04/28/creating-lmdb-in-python).
* Convolution can be rolled in to recurrent network to perform activity recognition, image description or video description. On [Jeff Donaue's website](http://jeffdonahue.com/lrcn/), one of the main contributors to Caffe, you can find the link to his published paper and video examples of a video description problem.

---

[Lab05](../lab05) &#124; [Introduction to Deeep Learning on ShARC Home](../) &#124;
