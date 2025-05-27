# Documentation for `src/Pardiso.H`

## Overview

This header file provides an interface to the Pardiso sparse linear solver. Pardiso is a high-performance library for solving large sparse systems of linear equations, available as part of Intel MKL or as a standalone package. This file declares C-style Fortran interface functions for Pardiso and C++ template functions within the `Pardiso` namespace to handle solver initialization, matrix type determination, equation solving, and (conditionally) sparse matrix inversion. It includes preprocessor directives to manage different Pardiso interface versions (e.g., `OLD_PARDISO_INTERFACE`).

## Key Components

### Namespaces
*   `Pardiso`: Encapsulates functions and utilities for interacting with the Pardiso solver.

### Functions

#### `extern "C"` Fortran Interface Functions
*   `void fortran_name(pardisoinit, PARDISOINIT)(...);`: Declaration for Pardiso initialization routine. The signature varies based on Pardiso interface version.
*   `int fortran_name(pardiso, PARDISO)(...);`: Declaration for the main Pardiso solver routine. The signature varies based on Pardiso interface version.

#### Functions within `Pardiso` namespace
*   `template <typename T> void sparse_solve(TCSR<T>* A, T* b, int b_cols, T* x);`
    *   **Description**: Solves a sparse linear system `Ax = b` using Pardiso. The matrix `A` is in `TCSR` format, `b` is the right-hand side (can have multiple columns `b_cols`), and `x` stores the solution. The implementation is provided inline in this header.
    *   **Doxygen Comment**: `/** \brief Function to solve a sparse linear system ... \pre The given matrix is a non-singular matrix ... \post x contains the solution ... */`
*   `template <typename T> void sparse_invert(TCSR<T>* A);`
    *   **Description**: (Conditionally compiled if `OLD_PARDISO_INTERFACE` is NOT defined) Performs a sparsity-preserving inversion of the matrix `A` (in-place) using Pardiso. The implementation is provided inline in this header.
    *   **Doxygen Comment**: `/** \brief Function to selectively invert a CSR matrix using Pardiso ... \pre The given matrix is a non-singular matrix ... \post The input matrix has been replaced by its inverse ... */`
*   `template <typename T> int get_matrix_type(TCSR<T>* A);`
    *   **Description**: A template function (specializations in `Pardiso.C`) that returns an integer code representing the matrix type (e.g., real unsymmetric, complex unsymmetric) as required by Pardiso's `mtype` parameter.

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for using Pardiso::sparse_solve
// TCSR<CPX> matrix_A(...); // Assume initialized
// CPX rhs_vector[N];      // Assume initialized
// CPX solution_vector[N];
// int num_rhs = 1;

// // Pardiso functions are typically called through the LinearSolver interface
// // or directly if Pardiso is the chosen solver.
// // This example shows direct conceptual usage:
// Pardiso::sparse_solve(&matrix_A, rhs_vector, num_rhs, solution_vector);
// // solution_vector now contains the solution.
```

## Dependencies and Interactions

### Included Files
*   `#include <omp.h>`
*   `#include "Types.H"`
*   `#include "CSR.H"`

### External Libraries
*   **Pardiso**: This header is an interface to the Pardiso library. The actual Pardiso library (e.g., from Intel MKL) must be linked.
*   **OpenMP**: Used for setting the number of threads for Pardiso (`iparm[2]`).

### Interactions with Other Project Components
*   This file declares and partly implements an interface to the Pardiso solver.
*   It is used by OMEN when Pardiso is selected as the linear solver (via the `LinearSolver<T>` interface).
*   It depends on `CSR.H` for the `TCSR<T>` sparse matrix format and `Types.H` for `CPX` and other types.
*   The `get_matrix_type` function, specialized in `Pardiso.C`, provides necessary type information to the Pardiso library.

---
*This documentation is auto-generated and may require further manual refinement.*
