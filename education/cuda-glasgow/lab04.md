---
layout: page
title: Introduction to CUDA - Lab 04
permalink: /education/cuda-glasgow/lab04/
---

# GPU Computing Workshop: Lab 04 #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

Get the Starting code from github by cloning the master branch of the CUDALab04 repository from the RSE-Sheffield github account. E.g. 
    
    git clone https://github.com/RSE-Sheffield/CUDALab04.git
    
This will check out all the starting code for you to work with.

## Exercise 01 ##

In order to find the maximum mark from a set of records, we will need to apply a parallel reduction. Rather than summing elements, the reduction will propagate the maximum value of two elements. This will allow us to exploit the GPUs parallelism to find a single value. For this first exercise, we will do this using a CUDA implementation of a reduction. Some code has already been provided for you to complete. Examine the function `maximumMark_CUDA` and the associated kernel `maximumMark_CUDA_kernel`. 

* 1.1	The purpose of the `maximumMark_CUDA_kernel` is for each thread block to reduce `THREADS_PER_BLOCK` values to a single value. Each thread within the block loads a single element into shared memory, reduce the values from shared memory (to avoid global memory reads) and finally the first thread in the block will write the reduced value to the results array (`d_reduce_marks`). The result of the executed kernel will be an array with a single reduced value for each thread block. 

To reduce the shared memory values, you must modify the provided loop which has `log2(THREADS_PER_BLOCK)` iterations as each iteration divides a stride value in half (`stride>>= 1`). You should ensure that only stride threads perform each iteration of the loop and that each participating thread reads two values, makes a comparison to find the maximum value and then writes this to a suitable shared memory location which allows the next iteration to function correctly. An example is shown in the below figure which demonstrates how this algorithms should work for a simplified case where the thread block size is `8` and the number of iterations is `log2(8)= 3`;


{: .center}
![Reduction Pattern](\static\img\cuda\reduction.png)
 

* 1.2	The result of exercise 1.1 is array with a single reduced value for each thread in the block. Rather than reduce these values on the GPU we will read them back to the CPU and reduce them serially. For small numbers of values, it is unlikely that the GPU will outperform the CPU as it will be massively underutilised. Modify the `maximumMark_CUDA` function so that you read back the results of the kernel and reduce the final number of marks with a `for` loop.
* 1.3	As you can see from the example, reducing values requires a large amount of CUDA code. Fortunately reduction is one of the parallel primitive algorithms which is implanted in the Thrust library. To perform a reduction using Thrust, complete the following tasks:
    * Create a `thrust::device_vector` of type `float`. You should pass through the pointer to the host data array and a pointer to the end of the host data array as arguments to the `device_vector` constructor.
    * Use a `thrust::reduce` call to find the maximum value by passing the following arguments: an iterator to the start of your thrust device vector, an iterator to the end of your thrust device vector, an initial float value of zero and a binary function operator (in this case `thrust::maximum<float>()`). *Note: A custom `my_maximum` structure has also been provided so that you can see how an `operator` can be defined to be used as a custom thrust binary operator.*
    
## Exercise 02 ##

Using the same `marks.cu` file, we will now use thrust to find the number of marks which are greater than `90%`. There are a number of ways in which we can do this by applying different algorithms. We will explore two options. For each you may need to refer to the Thurst API documentation for additional information on the specific function arguments;

[Thrust API Docs](http://thrust.github.io/doc/modules.html)

* 2.1	We can find the number of marks greater than `90%` by sorting all the marks and then finding the index of the first mark which is greater than 90. To implement this, complete the following in the `sortSplit_Thrust function`.
    * Create a thrust device vector 
    * Apply the `thrust::sort` algorithm to sort the data in place
    * Use a `thrust::find_if` algorithm to return an iterator at the location of the first mark of greater than `90%`. Using the `my_maximum` structure example you will need to implement your own binary operator with the following function signature:
    ```__host__ __device__ bool operator()(float x);```
    * Use `thrust::distance` to find the number of elements between the start of your thrust device vector and the returned iterator. Use this to return the correct value.
    
* 2.2	An alternative approach to finding marks greater than `90%` is to use a prefix sum to partition and re-reorder the data. Thrust provided an algorithms for doing this called `thrust::partition`. This algorithm will partition the data into the original array with all records meeting the condition at the front of the array and all records failing the condition at the end of the array.  The partition function returns an iterator which points to the end of the data records which pass the condition. Modify the partition_Thrust function to use the `thrust::partition` function to find the number of marks which exceed `90%`. *Hint: you can re-use the binary operator from the previous task.*

* 2.3	Increase the `NUMBER_LOOPS` so that you can get more accurate timing values to compare the two approaches.

## Exercise Solutions ##

The exercise solutions are available from the solution branch of the repository. To check these out either clone the repository using the branch command to a new directory as follows;

    git clone -b solutions https://github.com/RSE-Sheffield/CUDALab04.git
 
Alternately commit your changes and switch branch:

    git commit -m “my local changes to src files” 
    git checkout solutions

You will need to commit your local changes to avoid overwriting your changes when switching to the solutions branch. You can then return to your modified versions by returning to the master branch.


---

&#124; [Lab 03](../lab03) &#124; [CUDA Training Home](../) &#124; [Lab05](../lab05) 


