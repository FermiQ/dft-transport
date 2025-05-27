# Documentation for `src/FullInvert.H`

## Overview

This header file defines the `FullInvert` class. The primary purpose of this class is to compute the explicit inverse of a dense, distributed matrix and then use this inverse to update a density matrix `P`. The operations are performed using ScaLAPACK routines for distributed memory parallelism. The Doxygen comment `/*! \class Full \brief Interface for dense matrix inversion */` (though it refers to `Full` instead of `FullInvert`) indicates its role in matrix inversion.

The inversion process involves:
1.  Converting an input sparse matrix (`mat`) to a dense distributed format.
2.  Performing LU factorization (`pzgetrf`) of the dense matrix.
3.  Computing the matrix inverse from the LU factors (`pzgetri`).
4.  Transforming the inverse back to a local dense format.
5.  Using the elements of the inverted matrix to update the non-zero values of the output sparse matrix `P`.

## Key Components

### Classes/Structs
*   `class FullInvert`:
    *   **Description**: A class designed to perform explicit inversion of a dense distributed matrix and apply it to update another matrix.
    *   **Public Members (Functions)**:
        *   `FullInvert(TCSR<CPX>* mat, TCSR<double>* P, CPX factor, MPI_Comm solver_comm)`:
            *   **Description**: Constructor that performs the matrix inversion and update.
            *   **Parameters**:
                *   `mat`: Pointer to the input `TCSR<CPX>` matrix to be inverted (after conversion to dense).
                *   `P`: Pointer to the `TCSR<double>` matrix (likely a density matrix) to be updated with `factor * real(mat_inverse)`.
                *   `factor`: A complex scaling factor applied to the inverse before updating `P`.
                *   `solver_comm`: MPI communicator for distributed operations.
        *   `~FullInvert()`: Destructor. Currently empty.

### Functions
*   *No free functions are defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for FullInvert
// TCSR<CPX> matrix_to_invert; // Assume this is initialized
// TCSR<double> density_matrix_P; // Assume this is initialized with the correct sparsity pattern
// CPX scaling_factor = {1.0, 0.0};
// MPI_Comm current_communicator = MPI_COMM_WORLD;

// // ... Initialize matrix_to_invert and density_matrix_P ...

// FullInvert inverter(&matrix_to_invert, &density_matrix_P, scaling_factor, current_communicator);
// // density_matrix_P is now updated with factor * real(matrix_to_invert_inverse)
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`

### External Libraries
*   **ScaLAPACK**: Essential for distributed dense matrix operations like LU decomposition (`c_pzgetrf`), matrix inversion from LU factors (`c_pzgetri`), and matrix redistribution (`c_pzgemr2d`).
*   **BLACS**: Used by ScaLAPACK for managing the process grid (`Cblacs_gridinit`, `Cblacs_gridinfo`, `Cblacs_gridexit`).
*   **MPI**: Required by BLACS and ScaLAPACK.

### Interactions with Other Project Components
*   **`CSR.H` (`TCSR<T>`)**: Takes a `TCSR<CPX>` as input, which is converted to a dense matrix for inversion. Updates a `TCSR<double>` matrix.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX` (implicitly through `CSR.H`).
*   **`Blas.H`**: Uses BLAS/LAPACK wrapper functions (e.g., `c_descinit`, `c_numroc`) for ScaLAPACK setup and operations.
*   This class provides a specific mechanism (explicit full inversion) for calculating quantities related to the inverse of a matrix, which might be used in certain parts of the Green's function calculation or density matrix computation.

---
*This documentation is auto-generated and may require further manual refinement.*
