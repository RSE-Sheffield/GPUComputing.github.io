---
layout: page
title: Lab 06 Recurrent Neural Networks
permalink: /education/intro_dl_sharc_dgx1/lab06/
---

# Practical 6: Recurrent Neural Networks #

**Remember to be working from the root directory of DLTraining code sample throughout all practicals.**

In this lab we will create a simple text generation model that allows us to see the capabilities of the Long Short Term Memory (LSTM) units. It will include creating a HDF5 data set from raw sample text 'Alice in Wonderland'. The lab introduces 4 new layers, the `HDF5Data`, `LSTM`, `Embed` and `Dropout` layers.

## LSTM Layer ##

The `LSTM` is a layer  that takes in a sequence of input over time steps and can be used to generate a sequence of output over time.


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

The `Embed` layer transforms a one-hot encoded input of size `input_dim` in to a continous vector of size `num_output`.

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
      type: "uniform"
      min: -0.08
      max: 0.08
    }
  }
}
```

This layer will be used to embed our character input generated from the sample text.

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

A copy of 'Alice in Wonderland' is located at `data/wonderland.txt`. We will be using this to generate a HDF5 data set for our model.

Create a python file `create_text_gen_dataset.py` add the start importing packages that we'll need:

```
import h5py
import numpy as np
import json
```

Delclare out the input and output file paths:

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

Run the file to generate the data:

```
$ python create_text_gen_dataset.py
Raw text length:  144436
Total Characters:  50001
Total Vocab:  47
Text length:  50000  stream length:  200  numstreams:  250
Input data shape:  (200, 250)
Cont data shape:  (200, 250)
Target data shape:  (200, 250)
Dataset created
```

*Note that we have 47 unique characters in our dataset, we need to keep this in mind as we design our `Embed` and `SoftmaxWithLoss` layer.*

## Exercise 6.1 Implementing the model ##
The diagram below shows the network we'll be implementing:

{: .center}
![LSTM](/static/img/intro_dl_sharc_dgx1/text_gen_lstm.png)

The layer sequence is:
* `HDF5Data`
* `LSTM`
* `Dropout`
* `InnerProduct`
* `SoftmaxWithLoss`

Create a `text_gen.prototxt` file and implement the model.


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
max_iter: 4000

# snapshot intermediate results
snapshot: 2000
snapshot_prefix: "text_gen"

# solver mode: CPU or GPU
solver_mode: GPU

clip_gradients: 0.7
```

Start training the model. You should get one with loss of around `0.4`.

## Exercise 6.2 Deploy the model ##

Create a deployable model file called `text_gen_deploy.prototxt` by replacing `HDF5Data` layer with `Input` layers. Also replace `SoftmaxWithLoss` layer with just the `Softmax` layer.

Check with the file `code/lab06/text_gen_deploy.prototxt` to ensure your model is correct.

## Setting up an interactive predictor ##

Need to write this section up.

Check with the file `code/lab06/generate_text.py` to ensure your script is correct.

## Exercise 6.3 Adding additional LSTM layers ##
* Try changing the dropout values when training
* Try adding additional `LSTM` layers to your model.
  * You can check with `code/lab/06/text_gen_multilayer.prototxt` and `code/lab/06/text_gen_multilayer_solver.prototxt`
  * Don't forget to change the model paths in your `generate_text.py`

## Extra ##

---

[Lab05](../lab05) &#124; [Introduction to Deeep Learning on ShARC Home](../) &#124;
