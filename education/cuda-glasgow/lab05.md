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

A GPU implementation has already been provided. The performance critical (and hence timed) part of the implementation requires copying the input data to the device, performing the calculations and finally reading the data back form the device. The provided implementation is strictly synchronous in execution and uses only the default CUDA stream. It is possible to improve the performance by breaking the task down into smaller independent parts and using streams to ensure that the copy engines(s) and compute engine are kept busy with independent work. 

The asynchronous version that we implement will loop through each stream and perform a copy to the device, kernel execution and copy back from the device. Each stream will be responsible for a subset of the data (of size `SAMPLES` divided by `NUM_STREAMS`). Implement this version by completing the following tasks which requires you to modify the `cudaCalculatorNStream` function;

1.	Allocate the host and device memory so that it can be used for asynchronous copying.
2.	Create `NUM_STREAMS` streams in the streams array declared within the function.
3.	Loop through the streams and schedule each stream to perform a host to device memory copy on a suitable subset of the data.
4.	Loop through the streams and schedule each stream to perform the kernel execution on a suitable subset of the data.
5.	Loop through the streams and schedule each stream to perform a device to host memory copy on a suitable subset of the data. Hence copying back the result.
6.	Destroy each of the CUDA streams.
7.	Test and Benchmark your code. Modify the `NUM_STREAMS` value and complete the following table of results. Do you understand why the two different stream versions differ?

|Version | Streams | GPU timing (s) |
|---|---|---|
|cudaCalculatorDefaultStream | NA | |	
|cudaCalculatorNStream	| 2 | |
|cudaCalculatorNStream	| 4 | |	
|cudaCalculatorNStream	| 8 | |	
|cudaCalculatorNStream	| 16 | |



## Exercise Solutions ##

The exercise solutions are available from the solution branch of the repository. To check these out either clone the repository using the branch command to a new directory as follows;

    git clone -b solutions https://github.com/RSE-Sheffield/CUDALab05.git
 
Alternately commit your changes and switch branch:

    git commit -m “my local changes to src files” 
    git checkout solutions

You will need to commit your local changes to avoid overwriting your changes when switching to the solutions branch. You can then return to your modified versions by returning to the master branch.


---

&#124; [Lab 04](../lab04) &#124; [CUDA Training Home](../) &#124; 


