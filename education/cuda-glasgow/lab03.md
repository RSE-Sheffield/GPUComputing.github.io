---
layout: page
title: Introduction to CUDA - Lab 03
permalink: /education/cuda-glasgow/lab03/
---

# GPU Computing Workshop: Lab 03 #

*by Dr [Paul Richmond](http://paulrichmond.shef.ac.uk/) (University of Sheffield)*

Get the Starting code from github by cloning the master branch of the CUDALab03 repository from the RSE-Sheffield github account. E.g. 
    
    git clone https://github.com/RSE-Sheffield/CUDALab03.git
    
This will check out all the starting code for you to work with.

## Exercise 01 ##

{: .center}
![Matrix Multiply](\static\img\cuda\matrix_mul.png)

For exercise one, we are going to modify an implementation of matrix multiply (provided in `matrixmul.cu`). The implementation provided multiplies a Matrix A by a Matrix B to produce a Matrix C. The widths and heights of the matrices can be modified but for simplicity must be a factor of the `BLOCK_SIZE` (which has been predefined as a macro in the code). The implementation is currently very inefficient as it performs `A_WIDTH × B_HEIGHT` memory loads to compute each product (value in matrix C). To improve this, we will implement a blocked matrix multiply which uses CUDAs shared memory to reduce the number of memory reads by a factor of `BLOCK_SIZE`. First note the performance of the original version and then modify the existing code to perform a blocked matrix multiply.

To implement a blocked matrix multiply, it is required that we load `NUM_SUBS` square sub matrices of the matrix A and B into shared memory so that we can compute the intermediate result of the sub matrix products. In the example figure  (above), where the `NUM_SUBS` is equal to two, the sub matrix `C(1,1)` can be calculated by a square thread block of `BLOCK_SIZE x BLOCK_SIZE` where each thread (with location `tx`, `ty` in the square thread block) performs the following steps which requires two stages of loading matrix tiles into shared memory.


1. Load the two sub-matrices $$A_{1,0}$$ and $$B_{0,1}$$ into shared memory. For these sub matrices each thread should
    * Load an element of the sub matrix $$A_{1,0}$$ into shared memory from the matrix $$A$$ at position `(ty+BLOCK_SIZE, tx)`
    * Each thread should load an element of the sub matrix $$B_{1,0}$$ into shared memory from the matrix $$B$$ at position `(ty, tx+BLOCK_SIZE)`
    * Synchronise to ensure all threads have completed loading sub matrix values to shared memory
2. Compute the dot product of each row in sub-matrix $$A_{1,0}$$  with each column in the sub-matrix $$B_{0,1}$$, storing the result in a local variable. This is achieved through the following steps.
    * Iterate from `0` to `BLOCK_SIZE` to multiply row `ty` of $$A_{1,0}$$ (from shared memory) by column `tx` of $$B_{0,1}$$ (from shared memory) to calculate the sub matrix product value.
    * Store the sub matrix product value in a thread local variable.
    * Synchronise to ensure that all threads have finished reading from shared memory.
3.	Repeat steps 1 & 2 for the next sub-matrix (or matrices in the general case), adding each new dot product result to the previous result. For the example in the figure, there is only one more iteration required to load the final 2 sub matrices. E.g. 
    * Load an element of the sub matrix $$A_{1,1}$$ into shared memory from Matrix $$A$$ at position `(ty+BLOCK_SIZE, tx+BLOCK_SIZE)`
    * Load an element of the sub matrix $$B_{1,1}$$ into shared memory from matrix $$B$$ at position `(ty+BLOCK_SIZE tx+BLOCK_SIZE,)`
    * Synchronise to ensure all threads have completed loading sub matrix values to shared memory
    * Iterate from `0` to `BLOCK_SIZE` to multiply row `ty` of $$A_{1,1}$$ (from shared memory) by column `tx` of $$B_{1,1}$$ (from shared memory) to calculate the sub matrix product value.
    * Add this sub matrix product value to the one calculated for the previous sub matrices.
4. Store the sum of the sub-matrix dot products into global memory at location `(x, y)` of Matrix $$C$$.

Following the approach described above for the example in figure, modify the code where it is marked `TODO` to support the general case (any sizes of $$A$$ and $$B$$ which are a multiple of `BLOCK_SIZE`). Test and benchmark your code compared to the original version. 

*Note: When building in release mode the CUDA compiler often fuses a multiply and add instruction which causes an improvement in performance but some small loss of accuracy. To ensure that your test passes, you should either*

1.	Implement a small epsilon value and compare the difference of the host and device version to this to ensure that it is within an acceptable range.
2.	Modify the `nvcc` compiler flag to avoid the use of fused multiply by adding `–fmad=false` to your compilation options.

## Viewing the Images for Exercise 02 using X Forwarding ##

Assuming you have started your `ssh` session with the `–X` argument you can use X forwarding to view the image using the `viewraytrace.py` python file provided.  You will need to ensure that you have an X server running on your local machine.  You can run the following which will open a graphical window on your local machine displaying the image from the remote Amazon instance.
  
    python viewraytrace.py

## Exercise 02 ##

For this exercise, we are going to optimise a piece of code which implements a simple ray tracer. We will explore how changing the different types of memory affect performance. The ray tracer is a simple ray casting algorithm which casts a raw for each pixel into a scene consisting of sphere objects. The ray checks for intersections with the spheres, where there is an intersection, a colour value for the pixel is generated based on the intersection position of the ray on the sphere (giving an impression of forward facing lighting). For more information on the ray tracing technique, read Chapter 6 of the CUDA by Example book which this exercise is based on. Try compiling and executing the starting code `raytracer.cu` and examining the output image (`output.ppm`). 

The initial code places the spheres in GPU global memory. We know that there are two good options for improving this in the form of constant memory and read only memory. Implement the following changes.

* 2.1	Create a modified version of ray tracing kernel which uses the read-only data cache (`ray_trace_read_only`). You should implement this by using the `const` and `__restrict__` qualifiers on the function arguments. You will need to complete the kernel and kernel call. Calculate the execution time of the new version alongside the old version so that they can be directly compared: You will need to also create a modified version of the sphere intersect function (`sphere_intersect_read_only`).
* 2.2	Create a modified version of ray tracing kernel which uses the constant data cache (`ray_trace_const`). You will need to complete the kernel and kernel call. Calculate the execution time of the new version alongside the other two versions so that they can be directly compared.
* 2.3	How does the performance compare? Is this what you expected and why? Modify the number of spheres to complete the following table. For an extra challenge try to do this automatically so that you loop over all the sphere count sizes in the table and record the timing results in a 2D array. 

Sphere Count | Normal | Read-only cache | Constant cache 
--- | --- | --- | --- |
16 | | | 			
32 | | | 			
64 | | | 			
128 | | | 			
256 | | | 	
1024 | | | 			
2048 | | | 			

## Exercise Solutions ##

The exercise solutions are available from the solution branch of the repository. To check these out either clone the repository using the branch command to a new directory as follows;

    git clone -b solutions https://github.com/RSE-Sheffield/CUDALab03.git
 
Alternately commit your changes and switch branch:

    git commit -m “my local changes to src files” 
    git checkout solutions
 
You will need to commit your local changes to avoid overwriting your changes when switching to the solutions branch. You can then return to your modified versions by returning to the master branch.







---

&#124; [Lab 02](../lab02) &#124; [CUDA Training Home](../) &#124; [Lab04](../lab04) &#124;


