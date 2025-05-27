# Documentation for `src/ParallelEig.H`

## Overview

This header file declares a set of functions designed for performing parallel dense linear algebra operations, with a primary focus on solving generalized eigenvalue problems (`Ax = \lambda Bx`). These functions are intended to be used in a distributed memory environment and leverage the ScaLAPACK library for high-performance computations. The interface supports both real and complex matrices.

## Key Components

### Functions
*   `int p_grid_desc_init(int& icontxt, int nprocs, int nvec, int& rloc, int& cloc, int* descfull, int* descloc);`
    *   **Description**: Initializes a BLACS (Basic Linear Algebra Communication Subprograms) process grid and ScaLAPACK array descriptors, which are necessary for distributing matrices across processes.
*   `int p_eig(double* KSfull, double* OVfull, double* eigval, int nvec, MPI_Comm p_eig_comm);`
    *   **Description**: Solves the generalized eigenvalue problem for real, dense matrices, typically `KSfull * X = eigval * OVfull * X`.
*   `int p_eig(CPX* KSfull, CPX* OVfull, double* eigval, int nvec, MPI_Comm p_eig_comm);`
    *   **Description**: Solves the generalized eigenvalue problem for complex, dense matrices, typically for Hermitian matrices.
*   `int p_inv(CPX* Afull, int nvec, MPI_Comm p_eig_comm);`
    *   **Description**: Computes the inverse of a complex dense distributed matrix.
*   `int p_inv(CPX* Afull, CPX* Bfull, int nvec, CPX z, MPI_Comm p_eig_comm);`
    *   **Description**: Performs a parallel matrix operation that involves an inversion and scaling, likely related to Green's function calculations of the form `B = z*inv(A) - (1/z)*inv(A)^T` or similar.
*   `int p_lin(CPX* Afull, CPX* RHSfull, CPX* SOLfull, int nvec, int nrhs, MPI_Comm p_eig_comm);`
    *   **Description**: Solves a dense complex linear system `A * X = B` in parallel.
*   `int p_lin(CPX* Afull, CPX* RHSfull, int nvec, int nrhs, CPX z, MPI_Comm p_eig_comm);`
    *   **Description**: Solves dense complex linear systems involving `Afull` and its transpose with `RHSfull`, then combines results into `Afull` and `RHSfull` using a complex factor `z`.

### Macros
*   `LOGCERR`: Defined as `std::cerr << "ERROR IN LINE " << __LINE__ << " OF FILE " << __FILE__ << std::endl`. Used for logging error messages.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for solving a parallel eigenvalue problem:
// MPI_Comm comm = MPI_COMM_WORLD;
// int matrix_dim = 1000;
// double* H_matrix; // Assume distributed
// double* S_matrix; // Assume distributed
// double eigenvalues[matrix_dim];
// // ... Initialize H_matrix and S_matrix across MPI processes ...
// int error_status = p_eig(H_matrix, S_matrix, eigenvalues, matrix_dim, comm);
// if (error_status == 0) {
//     // Eigenvalues are now available in 'eigenvalues' on all processes.
//     // Eigenvectors typically overwrite H_matrix.
// }
```

## Dependencies and Interactions

### Included Files
*   `#include "ScaLapack.H"`

### External Libraries
*   **ScaLAPACK**: This is the core external library these functions are designed to interface with for distributed dense linear algebra.
*   **BLACS**: A dependency of ScaLAPACK, used for process grid management and communication.
*   **MPI**: The underlying message passing interface for ScaLAPACK and BLACS.

### Interactions with Other Project Components
*   This header file declares functions that are implemented in `ParallelEig.C`.
*   These functions provide high-level parallel dense linear algebra capabilities that can be utilized by other modules within OMEN requiring such operations (e.g., specific eigenvalue solvers for band structure calculations, or parts of direct solvers).
*   It depends on `Types.H` (via `ScaLapack.H`) for the `CPX` complex data type and the `LOGCERR` macro.

---
*This documentation is auto-generated and may require further manual refinement.*
