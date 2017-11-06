---
layout: page
title: Introduction to CUDA - Lab 01
permalink: /education/dundee/lab01/
---

# GPU Computing Workshop: Lab 01 #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

Get the Starting code from github by cloning the master branch of the CUDALab01 repository from the RSE-Sheffield github account. E.g. 
    
    git clone https://github.com/RSE-Sheffield/CUDALab01.git
    
This will check out all the starting code for you to work with.

## Exercise 01 ##

Exercise 1 requires that we de-cipher some encrypted text. The text provided in the file `encrypted01.bin` has been encrypted by using an affine cipher. The affine cypher is a very simple type of monoalphabetic substitution cypher where each numerical character of the alphabet is encrypted using a mathematical function. The encryption function is defined as;

$$E(x)=(Ax+B) mod M$$

Where $$A$$ and $$B$$ are keys of the cypher, mod is the modulo operation and $$A$$ and $$M$$ are co-prime. For this exercise the value of $$A$$ is `15`, $$B$$ is `27` and $$M$$ is `128` (the size of the ASCII alphabet). The affine decryption function is defined as:

$$D(x)= A^{-1} (x-B)  mod M$$

Where $$A^{-1}$$ is the modular multiplicative inverse of $$A modulo M$$. For this exercise $$A^{-1}$$ has a value of `111`. 

*Note: The mod operation is not the same as the remainder operator (`%`) for negative numbers. A suitable mod function has been provided for the example.*

As each of the encrypted character values are independent we can use the GPU to decrypt them in parallel. To do this we will launch a thread for each of the encrypted character values and use a kernel function to perform the decryption. Starting from the code provided in `exercise01.cu`, complete the following;

* 1.1. Modify the modulo function so that it can be called on the device by the `affine_decrypt` kernel. 
* 1.2. Implement the decryption kernel for a single block of threads with an `x` dimension of `N` (`1024`). The function should store the result in `d_output`. You can define the inverse modulus `A`, `B` and `M` using a C pre-processor definition. 
* 1.3. Allocate some memory on the device for the input (`d_input`) and output (`d_output`). 
* 1.4. Copy the host input values in `h_input` to the device memory `d_input`.
* 1.5. Configure a single block of `N` threads and launch the `affine_decrypt` kernel.
* 1.6. Copy the device output values in `d_output` to the host memory `h_output`.
* 1.7. Compile and execute your program. If you have performed the exercise correctly, you should decrypt the text.
* 1.8. Don’t go running off through the forest just yet! Modify your code to complete the `affine_decrypt_multiblock` kernel which should work when using multiple blocks of threads. Change your grid and block dimensions so that you launch `8` blocks of `128` threads.

## Exercise 02 ##

In exercise 2 we are going to extend the vector addition example from the lecture. The file `exercise02.cu` has been provided as a starting point. Perform the following modifications.

* 2.1. The code has an obvious mistake. Rather than correct it implement a CPU version of the vector addition (Called `vectorAddCPU`) storing the result in an array called `c_ref`. Implement a new function `validate` which compares the GPU result to the CPU result. It should print an error for each value which is incorrect and return a value indicating the total number of errors. You should also print the number of errors to the console. Now fix the error and confirm your error check code works.
* 2.2. Change the value of `N` to `2050`. Do not run your code yet as it will now perform unsafe writes beyond the memory bounds which you have allocated. This is because a whole thread block is required for the extra two threads (our grid is always made upt of entire blocks). You should modify the kernel by adding a check in the kernel so that you do not write beyond the bounds of the allocated memory. This will require you the ensure that the threads unique position that it indexed into memory does not exceed `N`. Threads which fail this test should no nothing. 

## Exercise 03 ##

We are going to implement a matrix addition kernel. In matrix addition, two matrices of the same dimensions are added entry wise. If you modify your code from exercise 2 by copying the file to a new file called `exercise03.cu`. It will require the following changes;

* 3.1. Modify the value of `size` so that you allocate enough memory for a matrix size of `N x N` and moves the correct amount of data using `cudaMemcpy`. Set `N` to `2048`. 
* 3.2. Modify the `random_ints` function to generate a random matrix rather than a vector.
* 3.3. Rename your CPU implementation to `matrixAddCPU` and update the validate function.
* 3.4. Change your launch parameters to launch a 2D grid of thread blocks with `256` threads per block. Create a new kernel (`matrixAdd`) to perform the matrix addition. *Hint: You might find it helps to reduce `N` to a single thread block to test your code.*
* 3.5. Finally modify your code so that it works with none square arrays of `N x M` for any size. 

## Exercise Solutions ##

The exercise solutions are available from the solution branch of the repository. To check these out either clone the repository using the branch command to a new directory as follows;

    git clone -b solutions https://github.com/RSE-Sheffield/CUDALab01.git
 
Alternately commit your changes and switch branch

    git commit -m “my local changes to src files” 
    git checkout solutions
 
You will need to commit your local changes to avoid overwriting your changes when switching to the solutions branch. You can then return to your modified versions by checking out the master branch.


---

&#124; [Getting Started](../qwiklab) &#124; [CUDA Training Home](../) &#124; [Lab02](../lab02) &#124;


