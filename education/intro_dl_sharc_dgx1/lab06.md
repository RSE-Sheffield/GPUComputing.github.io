---
layout: page
title: Lab 06 Recurrent Neural Networks
permalink: /education/intro_dl_sharc_dgx1/lab06/
---

# Practical 6: Recurrent Neural Networks #

Make in to LMDB
LMDB tutorial here http://deepdish.io/2015/04/28/creating-lmdb-in-python/

## Recurrent models ##
Temporal coherance
LSTM

### Running the model

pre-built ...

### Preparing the data


### Implementing the model

### Setting up an interactive predictor
Using the dropout layer to reduce overfitting
```
layer {
name: "drop1"
type: "Dropout"
bottom: "ip11"
top: "ip11"
dropout_param {
dropout_ratio: 0.5
}
}
```
---

&#124; [Lab05](../lab05) &#124; [Introduction to Deeep Learning on ShARC Home](../) &#124;
