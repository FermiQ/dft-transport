# Documentation for `src/Utilities.H`

## Overview

This header file declares a collection of miscellaneous utility functions and defines several template functions that are used across the OMEN project. These utilities cover a range of functionalities including:
*   Sorting helpers for custom data structures.
*   Numerical utility functions.
*   Helper functions for initializing and deleting arrays of CSR (Compressed Sparse Row) matrices.
*   Vector manipulation functions (min, max, sum, sort).
*   MPI environment validation.
*   Timing functions.
*   Random number generation.
*   Domain decomposition and related calculations.
*   Conditional CUDA initialization/finalization functions.

## Key Components

### Functions
*   `bool sortx(const XYZPOS&, const XYZPOS&)`: Sorts `XYZPOS` based on x-coordinate.
*   `bool sorty(const XYZPOS&, const XYZPOS&)`: Sorts `XYZPOS` based on y-coordinate.
*   `bool sortz(const XYZPOS&, const XYZPOS&)`: Sorts `XYZPOS` based on z-coordinate.
*   `bool sortn(const CONNEC&, const CONNEC&)`: Sorts `CONNEC` based on `neigh` member.
*   `bool sorti(const IJPOS&, const IJPOS&)`: Sorts `IJPOS` based on `i` member.
*   `bool sortj(const IJPOS&, const IJPOS&)`: Sorts `IJPOS` based on `j` member.
*   `bool my_isnan(double*,int)`: Checks for NaN in a double array.
*   `void dreshape(int, int, double*, int*)`: Reshapes/reorders a double array.
*   `void icopy(int,int*,int*)`: Copies an integer array.
*   `int Round(double)`: Rounds a double to the nearest integer.
*   `void init_CSR(CSR**,int,int,int)`: Initializes an array of `CSR` objects.
*   `void del_CSR(CSR**,int)`: Deletes an array of `CSR` objects.
*   `double min_vec(double*,int,int)`: Finds the minimum value in a double array.
*   `double max_sign_abs_vec(double*,int,int)`: Finds the value with max absolute magnitude, preserving sign.
*   `void change_sign(double*,int,int,int)`: Changes the sign of elements in a double array.
*   `void sort_vec(double*,int)`: Sorts a double array.
*   `void sort_vec(double*,int*,int)`: Sorts a double array and returns index order.
*   `void sort_abs_imag(CPX*,int*,int)`: Sorts a complex array by absolute imaginary part.
*   `void sort_abs_vec(CPX*,int*,int)`: Sorts a complex array by absolute value.
*   `void check_mpi(int,int,int,int,int,int,int,int)`: Validates MPI configuration.
*   `double get_time(double)`: Gets current time minus an offset.
*   `double randn()`: Generates a normally distributed random number.
*   `void domain_decomposition(int,int,int,int,int*,int*,int,int,int,int)`: Calculates domain decomposition for MPI ranks.
*   `int get_cpu_color(int,int,int,int)`: Determines CPU color for MPI communicator splitting.
*   `int get_number_of_blocks(int,int,int,int*)`: Calculates number of blocks.
*   `int get_msize(int,int,int,int*)`: Calculates matrix size.
*   `int get_max_orb(int*,int)`: Gets maximum number of orbitals.
*   `int sum_vec(int,int*,int*)`: Sums elements of an integer vector based on an activity flag.
*   `int min_active_vec(int,int*,int*)`: Finds index of min active element in an integer vector.
*   `int max_active_vec(int,int*,int*)`: Finds index of max active element in an integer vector.
*   `void cuda_initialize()`: Placeholder for CUDA initialization (likely enabled if `CUDA` is defined).
*   `void cuda_finalize()`: Placeholder for CUDA finalization (likely enabled if `CUDA` is defined).

### Template Functions
*   `template <typename T> void init_var(T *var,int N)`: Initializes an array of type `T` to zero.
*   `template <typename T> T sum_vec_comp(T *vec,int N,int inc)`: Sums components of a vector of type `T` with a given increment.
*   `template <typename T> T max_vec(T *vec,int N)`: Finds the maximum value in an array of type `T`.
*   `template <typename T> T max_vec(T *vec,int N,int inc)`: Finds the maximum value in an array of type `T` with a given increment.
*   `template <typename T> T max_vec(T *vec,int *ind,int N)`: Finds the maximum value in an array of type `T` using an index array.
*   `template <typename T,typename W> T convert(W val)`: A template for type conversion, with specializations for `double` and `CPX`.
    *   `double convert(CPX val)`: Returns `real(val)`.
    *   `CPX convert(CPX val)`: Returns `val`.
    *   `double convert(double val)`: Returns `val`.
    *   `CPX convert(double val)`: Returns `CPX(val,0.0)`.

### Classes/Structs
*   *None defined in this file, but it declares functions that operate on structs like `XYZPOS`, `CONNEC`, `IJPOS`, `CSR`, `CPX` (defined in `Types.H` and `CSR.H`).*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for init_var
// double my_array[10];
// init_var(my_array, 10); // Initializes all elements to 0.0

// Example for convert
// CPX c_val(5.0, 2.0);
// double d_val = convert<double>(c_val); // d_val will be 5.0
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <sys/time.h>`
*   `#include <sys/stat.h>`
*   `#include <sys/types.h>`
*   `#include <ctime>`
*   `#include <math.h>`
*   `#include <cmath>`
*   `#include "CSR.H"`
*   Commented out CUDA includes: `//#include <cuda.h>`, `//#include <cuda_runtime_api.h>`, `//#include <cublas.h>`, `//#include <cblas.h>`

### External Libraries
*   **MPI**: Used for `check_mpi` and `domain_decomposition`.
*   **CUDA**: Conditionally included, suggesting some functions might have CUDA-specific implementations or interactions if the `CUDA` macro is defined.

### Interactions with Other Project Components
*   This header file declares functions that are implemented in `Utilities.C`.
*   It provides commonly used utility functions and template utilities that are likely used by many other modules within the OMEN project.
*   Depends on `CSR.H` for the `CSR` type and `Types.H` (implicitly via `CSR.H` or through the types like `XYZPOS`, `CPX` used in function signatures) for various custom data structures.

---
*This documentation is auto-generated and may require further manual refinement.*
