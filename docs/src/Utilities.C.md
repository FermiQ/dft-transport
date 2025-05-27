# Documentation for `src/Utilities.C`

## Overview

This file provides a collection of miscellaneous utility functions used throughout the OMEN project. These include:
*   Sorting helper functions for `XYZPOS`, `CONNEC`, and `IJPOS` structures.
*   Numerical utilities such as a NaN (Not a Number) checker (`my_isnan`) and a custom rounding function (`Round`).
*   Memory management helper functions for arrays of `CSR` (Compressed Sparse Row) matrix objects (`init_CSR`, `del_CSR`).
*   Vector operations like finding minimum value (`min_vec`), maximum absolute value with sign (`max_sign_abs_vec`), changing sign of elements (`change_sign`), and sorting vectors of doubles or complex numbers (`sort_vec`, `sort_abs_imag`, `sort_abs_vec`).
*   An MPI environment check function (`check_mpi`) to validate CPU allocation for different parallelization levels.
*   A timing function (`get_time`) to measure execution time.
*   A function to generate normally distributed random numbers (`randn`).
*   Domain decomposition and block calculation helpers (`get_number_of_blocks`, `domain_decomposition`, `get_cpu_color`, `get_msize`, `get_max_orb`).
*   Utility functions for operating on integer vectors based on an activity flag (`sum_vec`, `min_active_vec`, `max_active_vec`).

## Key Components

### Functions
*   `bool sortx(const XYZPOS& a, const XYZPOS& b)`: Sorts `XYZPOS` objects based on the x-coordinate.
*   `bool sorty(const XYZPOS& a, const XYZPOS& b)`: Sorts `XYZPOS` objects based on the y-coordinate.
*   `bool sortz(const XYZPOS& a, const XYZPOS& b)`: Sorts `XYZPOS` objects based on the z-coordinate.
*   `bool sortn(const CONNEC& a, const CONNEC& b)`: Sorts `CONNEC` objects based on the `neigh` member.
*   `bool sorti(const IJPOS& a, const IJPOS& b)`: Sorts `IJPOS` objects based on the `i` member.
*   `bool sortj(const IJPOS& a, const IJPOS& b)`: Sorts `IJPOS` objects based on the `j` member.
*   `bool my_isnan(double *vec, int N)`: Checks if any element in a double array is NaN.
*   `void dreshape(int N, int inc, double *v, int *index)`: Reshapes/reorders a double array `v` based on `index`.
*   `void icopy(int N, int* x, int* x_copy)`: Copies an integer array `x` to `x_copy`.
*   `int Round(double x)`: Rounds a double to the nearest integer.
*   `void init_CSR(CSR **M, int size_i, int N, int nnz)`: Initializes an array of `CSR` matrix pointers.
*   `void del_CSR(CSR **M, int size_i)`: Deletes an array of `CSR` matrix objects.
*   `double min_vec(double *vec,int N,int inc)`: Finds the minimum value in a double array with a given increment.
*   `double max_sign_abs_vec(double *vec,int N,int inc)`: Finds the value with the maximum absolute magnitude in a double array, preserving its original sign.
*   `void change_sign(double *vec,int N,int inc,int sign)`: Multiplies elements of a double array by `sign`.
*   `void sort_vec(double *vec,int N)`: Sorts a double array in ascending order.
*   `void sort_vec(double *vec,int *idx_order,int N)`: Sorts a double array and returns the order of indices.
*   `void sort_abs_imag(CPX *vec,int *index,int N)`: Sorts a complex array based on the absolute value of the imaginary part, using an initial index mapping.
*   `void sort_abs_vec(CPX *vec,int *idx_order,int N)`: Sorts a complex array based on the absolute value of its elements and returns the order of indices.
*   `void check_mpi(int size,int rank,int CPU_per_vg_point,int CPU_per_vd_point,int CPU_per_kz_point, int Nky,int Nkz,int CPU_ppoint)`: Checks if the MPI configuration (total CPUs, CPUs per task) is valid for the given problem size (Nky, Nkz). Exits if invalid.
*   `double get_time(double t0)`: Returns the current wall time minus `t0`.
*   `double randn()`: Generates a normally distributed random number (using Box-Muller transform).
*   `int get_number_of_blocks(int NLayer,int layer_per_slab,int robust_numerics,int *neighbor_layer)`: Calculates the number of blocks based on layers and neighbor information.
*   `void domain_decomposition(int NLayer,int mpi_size,int NCS,int factor,int *IL_start,int *IL_stop, int spec_decomp,int CPU_per_solver,int CPU_per_bc,int interleave)`: Determines the start and stop indices for domain decomposition among MPI ranks.
*   `int get_cpu_color(int en_sub_group_rank,int interleave,int CPU_ppoint,int CPU_per_solver)`: Determines CPU color for MPI communicator splitting based on rank and interleaving strategy.
*   `int get_msize(int start,int stop,int tb,int* n_of_el)`: Calculates matrix size based on element counts and `tb` parameter.
*   `int get_max_orb(int* n_of_el,int NA)`: Gets the maximum number of orbitals for any atom in the list.
*   `int sum_vec(int N,int *vec,int *active)`: Sums elements of an integer vector where the corresponding `active` flag is 0.
*   `int min_active_vec(int N,int *vec,int *active)`: Finds the index of the minimum element in an integer vector where the corresponding `active` flag is 0.
*   `int max_active_vec(int N,int *vec,int *active)`: Finds the index of the maximum element in an integer vector where the corresponding `active` flag is 0.

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for sorting
// XYZPOS points[10];
// // ... populate points ...
// std::sort(points, points + 10, sortx);

// Example for timing
// double time_start = get_time(0.0);
// // ... code to measure ...
// double time_elapsed = get_time(time_start);
// printf("Elapsed time: %f seconds\n", time_elapsed);

// Example for MPI check
// check_mpi(num_procs, my_rank, cpus_vg, cpus_vd, cpus_kz, nky_val, nkz_val, cpus_pp);
```

## Dependencies and Interactions

### Included Files
*   `#include "Utilities.H"`
*   `#include "Blas.H"`

### External Libraries
*   **BLAS**: Implied by the inclusion of `Blas.H` and the use of BLAS-like functions `c_dcopy` and `c_zcopy`.
*   **MPI**: Used in `check_mpi` and `domain_decomposition` (`MPI_Barrier`, `MPI_COMM_WORLD`, `MPI_Allreduce`).

### Interactions with Other Project Components
*   This file implements the functions declared in `Utilities.H`.
*   It uses data structures defined in `Types.H` (e.g., `XYZPOS`, `CONNEC`, `IJPOS`, `CPX`).
*   It uses `CSR` objects (likely from `CSR.H`) in `init_CSR` and `del_CSR`.
*   The utility functions provided are likely used by many other modules within the OMEN project for common tasks.

---
*This documentation is auto-generated and may require further manual refinement.*
