---
layout: page
title: Introduction to CUDA - Lab 05
permalink: /education/cuda-glasgow/lab05/
---

# GPU Computing Workshop: Lab 05 (Advanced) #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

Get the Starting code from github by cloning the master branch of the CUDALab04 repository from the RSE-Sheffield github account. E.g. 
    
    git clone https://github.com/RSE-Sheffield/CUDALab05.git
    
This will check out all the starting code for you to work with.

## Exercise 01 ##


For this exercise we are going to modify the implementation of a command driven calculator. The calculators actions are a specified in a source file `command.calc` which can contain the following basic commands `add N`, `sub N`, `mul N`, `div N` and `exit`, where `N` is a floating point value. Each command is sequentially applied to an input value in parallel and hence our parallel calculated can apply the fixed set of commands on a large input data set. 

A GPU implementation has already been provided. The performance critical (and hence timed) part of the implementation requires copying the input data to the device, performing the calculations and finally reading the data back form the device. The provided implementation is strictly synchronous in execution and uses only the default CUDA stream. It is possible to improve the performance by breaking the task down into smaller independent parts and using streams to ensure that the copy engines(s) and compute engine are kept busy with independent work. We will implement 2 different streaming versions and compare the performance of each. 

The first asynchronous version that we implement will loop through each stream and perform a copy to the device, kernel execution and copy back from the device. Each stream will be responsible for a subset of the data (of size `SAMPLES` divided by `NUM_STREAMS`). Implement this version by completing the following tasks which requires you to modify the cudaCalculatorNStream1 function;

1.	Allocate the host and device memory so that it can be used for asynchronous copying.
2.	Create `NUM_STREAMS` streams in the streams array declared within the function.
3.	Create a loop to iterate the streams. For each stream launch stages 1, 2 and 3 from the synchronous version so that the stream operates on only a subset of the data. Test your code. 
4.	Destroy each of the CUDA streams.
For the second asynchronous version you can copy the allocation, stream creation and destruction code from your `cudaCalculatorNStream1` function to `cudaCalculatorNStream2`. For this exercise we will make a subtle change to the way work within streams is scheduled.
5.	Loop through the streams and schedule each stream to perform a host to device memory copy on a suitable subset of the data.
6.	Loop through the streams and schedule each stream to perform the kernel execution on a suitable subset of the data.
7.	Loop through the streams and schedule each stream to perform a device to host memory copy on a suitable subset of the data. Hence copying back the result.
8.	Test and Benchmark your code. Modify the `NUM_STREAMS` value and complete the following table of results. Do you understand why the two different stream versions differ?

|Version | Streams | GPU timing (s) |
|---|---|---|
|cudaCalculatorDefaultStream | NA | |	
|cudaCalculatorNStream1	| 2 | |
|cudaCalculatorNStream1	| 3 | |	
|cudaCalculatorNStream1	| 6 | |	
|cudaCalculatorNStream2	| 2 | |
|cudaCalculatorNStream2	| 3 | |	
|cudaCalculatorNStream2	| 6 | |	



## Exercise Solutions ##

The exercise solutions are available from the solution branch of the repository. To check these out either clone the repository using the branch command to a new directory as follows;

    git clone -b solutions https://github.com/RSE-Sheffield/CUDALab05.git
 
Alternately commit your changes and switch branch:

    git commit -m “my local changes to src files” 
    git checkout solutions

You will need to commit your local changes to avoid overwriting your changes when switching to the solutions branch. You can then return to your modified versions by returning to the master branch.


---

&#124; [Lab 04](../lab04) &#124; [CUDA Training Home](../) &#124; 


