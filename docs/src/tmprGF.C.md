# Documentation for `src/tmprGF.C`

## Overview

This file implements the `sparse_invert` function within the `tmprGF` namespace. The primary purpose of this function is to invert sparse block tridiagonal matrices using the rGF (recursive Green's function) algorithm. The inversion is performed in-place, meaning the input matrix is replaced by its inverse. The sparsity pattern of the input matrix is preserved in the output.

## Key Components

### Namespaces
*   `tmprGF`: Encapsulates the `sparse_invert` function.

### Functions
*   `void tmprGF::sparse_invert(TCSR<CPX> *matrix, std::vector<int> Bsizes)`:
    *   **Description**: Inverts a sparse block tridiagonal matrix in CSR format using the rGF algorithm. The operation is done in-place.
    *   **Parameters**:
        *   `matrix`: A pointer to the `TCSR<CPX>` matrix to be inverted.
        *   `Bsizes`: A `std::vector<int>` defining the sizes of the diagonal blocks.
    *   **Preconditions**: The input `matrix` must be non-singular.
    *   **Postconditions**: The input `matrix` is replaced by its inverse, maintaining the original sparsity pattern.

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None globally defined in this file. Variables like `GR`, `GRNNp1`, `T10_cur`, `T10_next` are local to the `sparse_invert` function and are used for intermediate calculations.*

## Usage Examples

```c++
// Example for sparse_invert (conceptual)
// TCSR<CPX> my_matrix; // Assume my_matrix is initialized
// std::vector<int> block_sizes; // Assume block_sizes is initialized

// // ... populate my_matrix and block_sizes ...

// tmprGF::sparse_invert(&my_matrix, block_sizes);
// // my_matrix now contains its inverse
```

## Dependencies and Interactions

### Included Files
*   `#include "tmprGF.H"`
*   `#include <cstddef>`
*   `#include <vector>`
*   `#include <algorithm>`    // For `std::swap` (C++98 version might be implied by comment)
*   `#include <utility>`      // For `std::swap` (C++11 version)

### External Libraries
*   *None directly identifiable from the includes in this specific file, beyond standard C++ libraries. The functionality relies on the `rGF` class.*

### Interactions with Other Project Components
*   **`tmprGF.H`**: Includes its own header file.
*   **`CSR.H` (`TCSR<CPX>`)**: Operates on sparse matrices of type `TCSR<CPX>`.
*   **`rGF.H` (`rGF` class)**: Utilizes an instance of the `rGF` class to perform the core recursive Green's function calculations (`myRGF->solve_blocks(...)`).
*   The function modifies the input `TCSR<CPX>` matrix directly.

---
*This documentation is auto-generated and may require further manual refinement.*
