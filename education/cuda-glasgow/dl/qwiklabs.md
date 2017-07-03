---
layout: page
title: Lab00 Getting Started
permalink: /education/cuda-glasgow/dl/qwiklabs/
---

# Lab 00: Introduction to Deep Learning With Caffe: Getting Started#

*by Twin Karmakharm (University of Sheffield)*

This training course will take place using the [NVIDIA Qwiklabs](https://nvlabs.qwiklab.com/) system. This is a cloud based training portal for GPU computing which uses Amazon AWS with GPU images. In order to access a GPU accelerated instance go to [NVIDIA quiklabs](https://nvlabs.qwiklab.com/) and create a free student account (Important: You must use the email address you provided when registering). Once created, you should be able to select the 'Introduction to CUDA and DL Glasgow' course, select `Deep Learning Lab` and then press `Select`. On the next screen press `Start lab` to launch your instance. After ~7 minutes the instance will start and you will be provided with connection information.

## Logging in to you Amazon Instance ##

The connection information will provide you with an Amazon cloud instance address and username password which can be accessed via an `ssh` terminal. To login from linux you can use the following command (note the requirement of -X for graphical window forwarding);

	ssh -X ubuntu@10-11-12-13.amazon.eu....

If you are in MacOSX then you can use the same command as above but you will need to install the X server ([more](https://support.apple.com/en-gb/HT201341)).

From Windows you can use the PuTTY program (if it is unavailable then you will need to install it). You will also need to install an X server. The free XMing application is recommended ([XMing](https://sourceforge.net/projects/xming/)). Within Putty, set the *“Host Name”* text field to your amazon instance e.g. `ubuntu@10-11-12-13.amazon.eu....`. To enable X windows forwarding ensure the *“Enable X Window Forwarding”* option is selected from the `Connection -> SSH -> X11` tab (see figure below). Select Open and login with the provided username and password.

{: .center}
![Putty configuration](\static\img\cuda\putty_config.png)


## Downloading lab materials ##

Clone the lab materials to your home directory from github:

```
git clone https://github.com/RSE-Sheffield/intro-to-dl-with-caffe.git ~/DLIntro
```

## Launching Docker Caffe ##

A convenient script is included in the materials for launching a Docker container that contains Caffe software that we will be using in this tutorial. Launch it like so:

```
~/DLIntro/rundocker.sh
```

It will take around 5 minutes for docker to download and extract the image. Once that's done you will automatically get a shell inside the container.

---

&#124; [Home](../../) &#124; [Lab01](../lab01) &#124;
