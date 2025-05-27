# Documentation for `src/Spike.C`

## Overview

This file provides template specializations for member functions of the `Spike` class, which is likely a class designed for linear algebra operations, possibly related to the SPIKE algorithm for solving banded or block-tridiagonal systems. The specializations cater to different data types, specifically `CPX` (complex double-precision) and `double` (real double-precision).

The implemented functions include:
*   Setting MPI data types corresponding to `CPX` or `double`.
*   Performing LU decomposition of dense matrices (`calculate_lu_decomposition`).
*   Solving linear systems using a pre-computed LU decomposition (`solve_linear_system_dense`).
*   Utility functions for visualizing matrices:
    *   `spy`: Prints a sparsity pattern of a matrix (dense or CSR).
    *   `full`: Prints the full content of a matrix (dense or CSR).
    *   `print_array`: Prints a 1D array.
*   Wrappers for BLAS/LAPACK routines like `xLACPY` (matrix copy) and `xGEMM` (matrix-matrix multiplication).

These implementations rely heavily on underlying BLAS and LAPACK routines, accessed through C-style wrapper functions (e.g., `c_zgetrf`, `c_dgemm`).

## Key Components

### Functions
*(These are template specializations of member functions of the `Spike<T>` class, where `T` is `CPX` or `double`)*

*   `void Spike<T>::set_mpi_dataype()`:
    *   **Description**: Sets the appropriate `MPI_Datatype` member variable (`MPI_data_type`) based on the template type `T`. `MPI_COMPLEX16` for `CPX` and `MPI_DOUBLE` for `double`.
*   `void Spike<T>::calculate_lu_decomposition(T* m, int rows, int cols)`:
    *   **Description**: Computes the LU decomposition of a dense matrix `m` using LAPACK routines (`c_zgetrf` for `CPX`, `c_dgetrf` for `double`). Stores pivot information in the `ipiv` member.
*   `void Spike<T>::solve_linear_system_dense(T *LU, int rows, int cols, T *B, int B_cols)`:
    *   **Description**: Solves a system of linear equations `A*X = B` given the LU factorization of `A` (stored in `LU`) and the right-hand side `B`. Uses LAPACK routines (`c_zgetrs` for `CPX`, `c_dgetrs` for `double`). The comment "M_TODO: since our diagonal blocks are actually symmetric..." suggests potential optimization.
*   `void Spike<T>::spy(T* matrix, int rows, int columns)`:
    *   **Description**: Prints a visual representation of the matrix, showing non-zero elements as colored asterisks and zero elements as plain asterisks. Useful for visualizing sparsity patterns of dense matrices.
*   `void Spike<CPX>::spy(TCSR<CPX>* matrix, int rows, int cols)`:
    *   **Description**: (Specialization for `CPX` and `TCSR<CPX>`) Prints a visual representation of a sparse CSR matrix, similar to the dense version.
*   `void Spike<T>::full(T* matrix, int rows, int columns)`:
    *   **Description**: Prints the full numerical values of a dense matrix, with non-zero elements highlighted. Output is formatted for readability.
*   `void Spike<CPX>::full(TCSR<CPX>* matrix, int rows, int cols)`:
    *   **Description**: (Specialization for `CPX` and `TCSR<CPX>`) Prints the full numerical values of a sparse CSR matrix.
*   `void Spike<CPX>::print_array(CPX* matrix, int size)`:
    *   **Description**: (Specialization for `CPX`) Prints the elements of a 1D array of complex numbers.
*   `void Spike<T>::xLACPY(char UPLO, int M, int N, T* A, int LDA, T* B, int LDB)`:
    *   **Description**: A wrapper for the LAPACK `DLACPY` or `ZLACPY` routine, which copies all or part of a matrix `A` to matrix `B`.
*   `void Spike<T>::xGEMM(char TRANSA, char TRANSB, int M, int N, int K, T ALPHA, T* A, int LDA, T* B, int LDB, T BETA, T* C, int LDC)`:
    *   **Description**: A wrapper for the BLAS `DGEMM` or `ZGEMM` routine, which performs matrix-matrix multiplication (`C = ALPHA*op(A)*op(B) + BETA*C`).

### Classes/Structs
*No new classes or structs are defined in this file. It provides template specializations for the `Spike<T>` class, which is declared in `Spike.H`.*

## Important Variables/Constants
*No global variables or constants are defined in this file. The `MPI_data_type` and `ipiv` variables are members of the `Spike<T>` class.*

## Usage Examples

```c++
// Conceptual example (assuming Spike object 'solver' is instantiated)

// For LU decomposition and solve (T can be CPX or double)
// T* matrix_A = ...; // Pointer to matrix data
// T* vector_B = ...; // Pointer to RHS data
// int n = ...; // Dimension of matrix A
// int nrhs = ...; // Number of columns in B

// solver.calculate_lu_decomposition(matrix_A, n, n);
// solver.solve_linear_system_dense(matrix_A, n, n, vector_B, nrhs);
// // vector_B now contains the solution X

// For matrix visualization
// solver.spy(matrix_A, n, n);
// solver.full(matrix_A, n, n);

// For BLAS/LAPACK wrappers
// T alpha = 1.0, beta = 0.0;
// solver.xGEMM('N', 'N', m, n, k, alpha, A_ptr, lda, B_ptr, ldb, beta, C_ptr, ldc);
```

## Dependencies and Interactions

### Included Files
*   `#include "Spike.H"`
*   `#include <iomanip>`      // For `std::setw`
*   `#include <sstream>`      // For `std::stringstream`
*   `#include <string>`       // For `std::string`

### External Libraries
*   **BLAS/LAPACK**: Essential for linear algebra operations. Indicated by calls to functions like `c_zgetrf`, `c_dgetrf`, `c_zgetrs`, `c_dgetrs`, `c_zlacpy`, `c_dlacpy`, `c_zgemm`, `c_dgemm`. These `c_` prefixed functions are likely C wrappers to Fortran BLAS/LAPACK routines.
*   **MPI**: Implied by the `set_mpi_dataype()` method and the `MPI_Datatype` member, suggesting use in a parallel environment.

### Interactions with Other Project Components
*   This file directly implements parts of the `Spike` class template, which is declared in `Spike.H`.
*   It relies on types like `CPX` (complex number type) and `TCSR` (sparse matrix type), which are likely defined in other project headers (e.g., `Types.H`, `CSR.H`).
*   The `c_` prefixed BLAS/LAPACK wrappers suggest an interaction with a lower-level module providing these C interfaces to Fortran libraries (potentially `Blas.H` or a similar utility).

---
*This documentation is auto-generated and may require further manual refinement.*
