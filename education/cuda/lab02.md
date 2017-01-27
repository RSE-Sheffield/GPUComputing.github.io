---
layout: page
title: Introduction to CUDA - Lab 02
permalink: /education/cuda/lab02/
---

# GPU Computing Workshop: Lab 02 #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

Get the Starting code from github by cloning the master branch of the CUDALab02 repository from the RSE-Sheffield github account. E.g. 
    
    git clone https://github.com/RSE-Sheffield/CUDALab02.git
    
This will check out all the starting code for you to work with.

## Introduction ##

For this session we are going to start by improving the performance of an existing CUDA program “boxblur.cu”. The starting code provided contains an implementation of a simple box blur. The box blur (also known as a box linear filter) is an operation which samples neighbouring pixels of an input image to output an average value. When applied iteratively to an image, the box filter can be used to approximate a more complicated Gaussian blur ([wiki link](https://en.wikipedia.org/wiki/Gaussian_blur)). The box blur can be described as follows.

$$Out_{x,y}=\frac{I_{x-1,y-1} +I_{x,y-1} +I_{x+1,y-1} +I_{x-1,y} +I_{x,y} +I_{x+1,y} +I_{x-1,y+1} +I_{x,y+1} +I_{x+1,y+1}}{9} $$

Within the implementation provided, the box blur has the property that outside of the bounds of the input image values are `0`. The code works for fixed sized square images. An image `input.ppm` is provided in the ppm format and code is provided for image reading and writing. You can use your own image but make sure that the `IMAGE_SIZE` macro is changed to reflect your image size.
   
{: .center}
![Box Blur](\static\img\cuda\blur.png)
*Figure 1 - Result of applying the Box filter for 0, 50 and 100 iterations*

Try compiling and running the code and examine the output of the blurred image. Make a note of the execution time reported.

## Viewing the Images using X Forwarding ##

Assuming you have started an interactive session on a CPU worker node with `qrshx` and your ssh session with the `–X` argument (in the Putty configurations), you can use X forwarding to view the image using the `viewinput.py` and `viewoutput.py` python files provided.  You will need to ensure that you have an X server running on your local (not ShARC) machine. If you are using a CICS managed desktop machine then run `Xming`. First run the following command from your interactive session to enable to necessary python imaging libraries;

    module load apps/python/anaconda3-4.2.0
 
Next you can run the following which will open a graphical window on your local machine displaying the image from the remote (ShARC) machine.
 
    python viewinput.py

{: .center}
![XMing](\static\img\cuda\xming.png)

## Exercise 01 ##

The code has a number of inefficiencies. We will first consider the transfer bottleneck. For each iteration of applying the box filter/blur the algorithms performs the following steps

1. Copy the previous iterations (or input) image from the host to the device
2. Apply the box blur GPU kernel
3. Copy the results back to the host and repeat the above.

It is not necessary to copy the results of each filter operation back to the host. We can simply pass the pointer of the previous iterations output as the input for the next iterations. This will drastically reduce memory movements via PCIe. To implement pointer swapping complete the following steps.

* 1.1	Starting from the code in the `STARTING_CODE` switch case make a copy into the `EXERCISE_01` switch case
* 1.2	Move the memory copy of the input image outside of the `ITERATIONS` loop so that the host data is copied to the device only once.
* 1.3	A pointer `d_image_temp` has been defined for you. Use this as a temporary pointer to swap the areas of memory pointer to by `d_image` and `d_image_output` after the box blur kernel is applied.
* 1.4	Move the memory copy of the output image outside of the `ITERATIONS` loop so that the device data is copied back to the host only once. *Note: Be careful that you copy back from the correct device pointer if you have swapped them!*
* 1.5	Compile and execute your code.  Ensure that the variable exercise is set to `EXERCISE_01` so that your modified code is executed. Make a note of the execution time. It should be considerably faster than previously.

## Exercise 02 ##

The `image_blur_columns kernel` currently has a poor memory access pattern. Let us consider why this is. For each thread which is launched the thread iterates over a unique row of `IMAGE_DIM` pixels to perform the blurring on each pixel. Between each thread this creates a stride of `IMAGE_DIM` between memory loads. CUDA code is much more efficient when sequential threads read from sequential values in memory (memory coalescing). To improve the code, we can implement a row wise version on the kernel by completing the following.

* 2.1	Copy the `image_blur_columns` kernel and call the new kernel `image_blur_rows`.
* 2.2	Modify the `image_blur_rows` kernel so that each thread operates on a unique column (rather than row of the images). This will ensure that sequential threads read sequential row values from memory.
* 2.3	Implement the `EXERCISE_02` switch case (by copying the previous one) ensuring that your host code calls your new kernel
* 2.4	Compile and execute your code. Ensure that the variable exercise is set to `EXERCISE_02` so that your modified code is executed. Make a note of the execution time. It should be considerably faster than previously.

## Exercise 03 ##

Our previous implementations of the blur kernel have a limited amount of parallelism. There are in total `IMAGE_DIM `threads launched and each of the threads is responsible for calculating a unique row or column. Whilst this number of threads might seem reasonably large it is unlikely that it is sufficient to occupy all of the Streaming Multiprocessors of the device. To increase the level of parallelism and improve the occupancy it is possible to launch a unique thread for each pixel of the image. To implement this, complete the following steps.

* 3.1	Make a copy of the image_blur_rows kernel and call it `image_blur_2d`. Modify the new kernel so that the `x` and `y` locations are determined from the thread and block index. You can then remove the row loop as the kernel is responsible for calculating only a single pixel value.
* 3.2	Implement the `EXERCISE_03` switch case (by copying the previous one). You will need to change the block and grid dimensions so that they launch `IMAGE_DIM²` threads in total.
* 3.3	Compile and execute your code. Ensure that the variable `exercise` is set to `EXERCISE_03` so that your modified code is executed. Make a note of the execution time. It should be considerably faster than previously.


## Exercise Solutions ##


The exercise solutions are available from the solution branch of the repository. To check these out either clone the repository using the branch command to a new directory as follows;
 
    git clone -b solutions https://github.com/RSE-Sheffield/CUDALab02.git

Alternately commit your changes and switch branch:

    git commit -m “my local changes to src files” 
    git checkout solutions

You will need to commit your local changes to avoid overwriting your changes when switching to the solutions branch. You can then return to your modified versions by returning to the master branch.


---

&#124; [Lab 01](../lab01) &#124; [CUDA Training Home](../) &#124; [Lab03](../lab03) &#124;


