# Documentation for `src/CSR.C`

## Overview

This file provides the implementation for the `CSR` class, which represents a sparse matrix using the Compressed Sparse Row (CSR) format. This particular implementation is a non-template version that stores the real and imaginary parts of complex matrix elements in separate `double` arrays (`r_nnz`, `i_nnz`). It includes functionalities for constructing and destructing CSR matrices, updating their diagonal elements, and writing them to a file.

## Key Components

### Functions
*(These are member functions of the `CSR` class, declared in `CSR.H`)*
*   `CSR(int N, int n_nnz)`:
    *   **Description**: Constructor for the `CSR` class. Allocates memory for a sparse matrix of size `N` (number of rows/columns if square) with `n_nnz` non-zero elements.
*   `~CSR()`:
    *   **Description**: Destructor for the `CSR` class. Frees the dynamically allocated memory for the matrix components.
*   `void update_diag(double *r_diag, double *i_diag)`:
    *   **Description**: Adds values from `r_diag` and `i_diag` to the real and imaginary parts of the diagonal elements of the matrix, respectively.
*   `void r_update_diag(double *r_diag)`:
    *   **Description**: Adds values from `r_diag` to the real part of the diagonal elements of the matrix.
*   `void i_update_diag(double *i_diag)`:
    *   **Description**: Adds values from `i_diag` to the imaginary part of the diagonal elements of the matrix.
*   `void get_row_edge()`:
    *   **Description**: Calculates and populates the `edge_i` array. `edge_i[k]` stores the index in `r_nnz`/`i_nnz`/`index_j` where the non-zero elements of row `k` begin. `edge_i[0]` is 0, and `edge_i[size]` is `n_nonzeros`. This relies on `index_i` (number of non-zeros per row) being pre-populated.
*   `void write(const char* filename)`:
    *   **Description**: Writes the sparse matrix to a text file in a coordinate format (row_index, column_index, real_value, imaginary_value). Row and column indices are written as 1-based.

### Classes/Structs
*   *None defined in this file. It implements methods for the `CSR` class declared in `CSR.H`.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for creating and using a CSR matrix (conceptual)
// int num_rows = 10;
// int num_non_zeros = 20;
// CSR my_matrix(num_rows, num_non_zeros);

// // ... populate my_matrix.index_i, my_matrix.index_j, 
// //     my_matrix.r_nnz, my_matrix.i_nnz ...
// my_matrix.get_row_edge(); // Important to call after setting index_i
// my_matrix.write("matrix_output.txt");

// double r_diag_update[num_rows];
// double i_diag_update[num_rows];
// // ... populate r_diag_update, i_diag_update ...
// // Assumes my_matrix.diag_pos is correctly set
// my_matrix.update_diag(r_diag_update, i_diag_update);
```

## Dependencies and Interactions

### Included Files
*   `#include <math.h>`
*   `#include "CSR.H"`

### External Libraries
*   *None directly identifiable from the includes, relies on standard C++ libraries.*

### Interactions with Other Project Components
*   This file implements the methods of the `CSR` class, which is declared in `CSR.H`.
*   The `CSR` class, due to its fundamental nature for representing sparse matrices, is likely used by various numerical algorithms, solvers, and data handling modules within the OMEN project, especially those that might predate or not use the templated `TCSR<T>` version.

---
*This documentation is auto-generated and may require further manual refinement.*
