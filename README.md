CUDA implementation of the Jacobi method
========================================

This project has been created in the scope of ["Introduction to GPU and accelerator programming for scientific computing"](http://sese.nu/introduction-to-gpu-and-accelerator-programming-for-scientific-computing-2014/), a course organized by [SeSE](http://sese.nu/) and [PDC at KTH](https://www.pdc.kth.se/).

Problem
-------

The Jacobi method is used to find approximate numerical solutions for systems of linear equations of the form *Ax = b*. The algorithm starts with an initial estimate for *x* and iteratively updates it until convergence. The Jacobi method is guaranteed to converge if the matrix *A* is diagonally dominant. 

Two problem sizes were considered in the scope of this assignment. The small problem consists of a randomly generated 512x512 coefficient matrix *A* and a 512x1 right hand side (rhs) vector *b*. The large problem is given by a randomly generated 2048x2049 coefficient matrix *A* and a 2048x1 rhs vector *b*.


Implementation
--------------

The given task was to implement the Jacobi method in several versions: a serial CPU function, an un-optimized CUDA kernel, and an optimized version of the CUDA kernel. The Jacobi method was implemented in C according to the pseudocode on [Wikipedia](http://en.wikipedia.org/wiki/Jacobi_method). The un-optimized kernel runs the inner loop in each thread where all threads belong to a single block. Two adjustments were then made to optimize this kernel. First, the index was precomputed and stored in a local register. This avoids one multiplication within each iteration in each thread. Second, the threads were tiled into blocks. This enables scaling to larger problem sizes. Prefetching and loop enrollment were also tested, but unfortunately it was not possible to reproduce correct results. Therefore, the last two approaches remain as comments in the code, but are not used.

The file *jacobi.cu* contains the CPU implementations of the Jacobi method as well as both CUDA kernels. In order to compile the program on zorn, the command `./install.sh` needs to be executed. This loads the necessary CUDA module *cuda/5.5* and executes the Makefile. The program can now be executed by `./jacobi <arguments>`. The following command line arguments are available:

    -h  --help			Display usage information.
    -f  --file filename		File containing coefficient matrix and right hand side (required).
    -i  --Ni int		Number of elements in Y direction (optional, default=512).
    -j  --Nj int		Number of elements in X direction (optional, default=512).
    -n  --iterations int	Number of iterations (optional, default=10000).
    -k  --kernel int		1: unoptimized, 2: optimized kernel (optional, default=2).
    -t  --tilesize int		Size of each thread block in kernel 2 (default=4).

The command `qsub job_512.sh` submits an example run on the small problem with default parameters to the queue of zorn. The large problem can be run with `qsub job_2048.sh`.

Input matrices of a particular size can be generated by `python gen_diag_dominant_matrix.py <size> <output_filename>`. A matrix and its corresponding rhs are created and stored in a text file, where each line contains one element of the row-majored matrix. The rhs is added to the end of the file. The small and large example problems are given by `test_512` and `test_2048`, respectively.


Benchmark
---------

All implementations were tested on the small and large problem sizes. The timings were recorded for 10,000 iterations of the Jacobi method.

**Host implementation:** The runtime for the serial CPU implementation is 17.25s for the small and 279.46s for the large problem.

**GPU implementation basic:** The un-optimized kernel needed 14.46s on the small problem. Unfortunately it was not possible to solve the large problem with this kernel, because the number of threads that would be required exceeded the number of available threads on the GPU.

**GPU implementation optimized:**  By storing the index in a register variable the runtime could be reduced to 9.46s on the small problem. Tiling was then used to be able to compute the large problem and for further runtime optimization. This resulted in an optimal runtime of 1.65s on the small problem with a tile size of 3. The large problem took 16.17s to compute with an optimal tile size of 4. [Table 1](#table1) shows all runtimes for the tested tile sizes and both problems.

<table name=table1 border="1">
<caption align=left>Table 1: Performance of the optimized kernel.</caption>
  <tr>
    <th></th>
    <th colspan="2">Runtime (s)</th>
  </tr>
  <tr>
    <td>Tile size</td>
    <td>Small (512)</td>
    <td>Large (2048)</td>
  </tr>
  <tr>
    <td>2</td>
    <td>2.09</td>
    <td>27.36</td>
  </tr>
  <tr>
    <td>3</td>
    <td>1.65</td>
    <td>20.91</td>
  </tr>
  <tr>
    <td>4</td>
    <td>2.29</td>
    <td>16.17</td>
  </tr>
  <tr>
    <td>5</td>
    <td>1.91</td>
    <td>17.45</td>
  </tr>
  <tr>
    <td>8</td>
    <td>2.61</td>
    <td>18.62</td>
  </tr>
  <tr>
    <td>16</td>
    <td>2.43</td>
    <td>20.57</td>
  </tr>
  <tr>
    <td>32</td>
    <td>4.36</td>
    <td>20.94</td>
  </tr>
  <tr>
    <td>64</td>
    <td>4.38</td>
    <td>20.33</td>
  </tr>
  <tr>
    <td>128</td>
    <td>4.39</td>
    <td>16.43</td>
  </tr>
  <tr>
    <td>256</td>
    <td>7.85</td>
    <td>29.41</td>
  </tr>
  <tr>
    <td>512</td>
    <td>15.88</td>
    <td>58.11</td>
  </tr>
</table>

