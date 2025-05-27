# Documentation for `src/CSC.H`

## Overview

This header file defines the `TCSC<T,W>` class template, which implements the Compressed Sparse Column (CSC) matrix format. CSC is an efficient way to store sparse matrices, where only non-zero elements are stored along with their row indices and pointers to the start of each column. This class provides constructors (including one to convert from CSR format), a destructor, methods for writing the matrix to a file, and matrix-vector multiplication routines. The template parameters `T` and `W` allow for flexibility in the data type of matrix elements and indices, respectively.

## Key Components

### Classes/Structs
*   `template <class T, class W> class TCSC`:
    *   **Description**: Represents a sparse matrix in Compressed Sparse Column (CSC) format.
    *   **Template Parameters**:
        *   `T`: The data type of the non-zero matrix elements (e.g., `double`, `CPX`).
        *   `W`: The data type for matrix indices (e.g., `int`, `long int`).
    *   **Public Members (Variables)**:
        *   `int size`: Number of columns in the matrix (also often the number of rows if square).
        *   `int n_nonzeros`: Number of non-zero elements in the matrix.
        *   `int findx`: Fortran-style indexing flag (1 if Fortran-based, 0 if C-based).
        *   `T *nnz`: Pointer to an array storing the non-zero values of the matrix.
        *   `W *index_i`: Pointer to an array storing the row indices of the corresponding non-zero values.
        *   `W *index_j`: Pointer to an array, usage seems to be for temporarily storing column counts during CSR to CSC conversion or for `get_column_edge()`.
        *   `W *edge_j`: Pointer to an array where `edge_j[col]` stores the index in `nnz` (and `index_i`) of the first non-zero element in column `col`. `edge_j[size]` stores `n_nonzeros`.
    *   **Public Members (Functions)**:
        *   `TCSC(int N, int n_nnz, int fortran_index)`: Constructor that allocates memory for a CSC matrix of `N` columns and `n_nnz` non-zero elements.
        *   `TCSC(TCSR<T>* CSRMat, int fortran_index)`: Constructor that converts a `TCSR` (Compressed Sparse Row) matrix to CSC format.
        *   `~TCSC()`: Destructor, frees allocated memory.
        *   `void write(const char* filename)`: Writes the matrix to a file in a sparse triplet format (row, column, value). Specializations for `double` and `CPX` with `int` or `long int` indices.
        *   `void init_variable(T* var, int N)`: Initializes an array `var` of size `N` with zeros.
        *   `void get_column_edge()`: Populates `edge_j` based on counts in `index_j`.
        *   `void vec_mat_mult(T* in, T* out, int NR)`: Performs matrix-vector multiplication `out = Matrix * in`. `NR` seems to indicate number of right-hand sides or block size if `in` and `out` represent multiple vectors. Specializations for `double,int` and `CPX,int`.
        *   `void vec_mat_mult_add(T* in, T* out, int NR)`: Performs matrix-vector multiplication and addition `out = out + Matrix * in`. Specializations for `double,int` and `CPX,int`.
        *   `void convert_csr(TCSR<T>* CSRMat)`: Converts a `TCSR` matrix to CSC format.
        *   `void convert_csr(TCSR<T>* CSRMat, int nrow, int ncol)`: Converts a `TCSR` matrix to CSC format, allowing specification of dimensions.
        *   `void imag_transpose(TCSR<double>* A, CPX factor)`: (Specialized for `TCSC<CPX,int>`) Computes `A = imag(factor * this^T)`, where `this` is the CSC matrix and `A` is a CSR matrix.
        *   `void add_imag_transpose(TCSR<double>* A, CPX factor)`: (Specialized for `TCSC<CPX,int>`) Computes `A = A + imag(factor * this^T)`.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for creating a CSC matrix from CSR
// TCSR<CPX> csr_matrix(...); // Assume csr_matrix is initialized
// TCSC<CPX, int> csc_matrix(&csr_matrix, 0); // Convert CSR to CSC with C-style indexing

// Example for matrix-vector multiplication
// CPX input_vector[N];
// CPX output_vector[N];
// // ... initialize input_vector ...
// csc_matrix.vec_mat_mult(input_vector, output_vector, 1);
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "Types.H"`

### External Libraries
*   *None directly identifiable from includes, primarily uses standard C++ and custom project types.* BLAS-like operations (`c_zaxpy`, `c_daxpy`, etc.) are used in member functions, implying a dependency on `Blas.H` for those implementations.

### Interactions with Other Project Components
*   **`CSR.H` (`TCSR<T>`)**: `TCSC` can be constructed from a `TCSR` matrix, and some methods also interact with `TCSR` objects.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX` and other fundamental types.
*   **`Blas.H`**: The `vec_mat_mult` and `vec_mat_mult_add` methods use BLAS-like helper functions (e.g., `c_zaxpy`, `c_daxpy`) which are likely defined in `Blas.H`.
*   This class provides a fundamental sparse matrix storage format that can be used by various numerical algorithms and solvers within the OMEN project.

---
*This documentation is auto-generated and may require further manual refinement.*
