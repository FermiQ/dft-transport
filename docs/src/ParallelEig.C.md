# Documentation for `src/ParallelEig.C`

## Overview

This file implements a set of functions for performing parallel linear algebra operations, primarily focused on solving dense generalized eigenvalue problems (`Ax = \lambda Bx`) and related tasks like matrix inversion and solving linear systems. These functions are designed to work in a distributed memory environment using MPI and leverage the ScaLAPACK library for high-performance computations. Both real (`double`) and complex (`CPX`) matrix types are supported for eigenvalue problems.

## Key Components

### Functions
*   `int p_grid_desc_init(int &icontxt, int nprocs, int nvec, int &rloc, int &cloc, int *descfull, int *descloc)`:
    *   **Description**: Initializes a BLACS process grid and ScaLAPACK array descriptors required for distributing matrices.
    *   **Parameters**:
        *   `icontxt`: (Output) BLACS context handle.
        *   `nprocs`: Number of MPI processes.
        *   `nvec`: Dimension of the square matrices.
        *   `rloc`, `cloc`: (Output) Local number of rows and columns for the current process.
        *   `descfull`: (Output) ScaLAPACK descriptor for the full (non-distributed locally) matrix.
        *   `descloc`: (Output) ScaLAPACK descriptor for the locally distributed part of the matrix.
    *   **Returns**: 0 on success, non-zero on error.
*   `int p_eig(double *KSfull, double *OVfull, double *eigval, int nvec, MPI_Comm p_eig_comm)`:
    *   **Description**: Solves the dense generalized eigenvalue problem `KSfull * X = eigval * OVfull * X` for real symmetric matrices `KSfull` (Kohn-Sham like) and `OVfull` (Overlap like). Eigenvectors `X` overwrite `KSfull`.
    *   **Parameters**:
        *   `KSfull`: (Input/Output) Pointer to the full Kohn-Sham like matrix. Overwritten by eigenvectors.
        *   `OVfull`: (Input) Pointer to the full Overlap like matrix.
        *   `eigval`: (Output) Array to store computed eigenvalues.
        *   `nvec`: Dimension of the matrices.
        *   `p_eig_comm`: MPI communicator for this operation.
    *   **Returns**: 0 on success, non-zero on error. Uses ScaLAPACK routine `pdsygvx`.
*   `int p_eig(CPX *KSfull, CPX *OVfull, double *eigval, int nvec, MPI_Comm p_eig_comm)`:
    *   **Description**: Solves the dense generalized eigenvalue problem for complex Hermitian matrices. Eigenvectors `X` overwrite `KSfull`.
    *   **Parameters**: Similar to the real version, but for `CPX` matrices.
    *   **Returns**: 0 on success, non-zero on error. Uses ScaLAPACK routine `pzhegvx`.
*   `int p_inv(CPX *Afull, int nvec, MPI_Comm p_eig_comm)`:
    *   **Description**: Computes the inverse of a complex dense distributed matrix `Afull` in place.
    *   **Parameters**:
        *   `Afull`: (Input/Output) Pointer to the full matrix to be inverted. Overwritten by its inverse.
        *   `nvec`: Dimension of the matrix.
        *   `p_eig_comm`: MPI communicator.
    *   **Returns**: 0 on success, non-zero on error. Uses ScaLAPACK routines `pzgetrf` (LU factorization) and `pzgetri` (inversion from LU factors).
*   `int p_inv(CPX *Afull, CPX *Bfull, int nvec, CPX z, MPI_Comm p_eig_comm)`:
    *   **Description**: Performs a sequence of operations: inverts `Afull` (storing it back in `Afull`), then computes `Bfull = z*inv(Afull) - (1/z)*inv(Afull)^T`. The exact formula might need verification based on context.
    *   **Parameters**:
        *   `Afull`: (Input/Output) Matrix to be inverted.
        *   `Bfull`: (Output) Resulting matrix.
        *   `nvec`: Dimension of matrices.
        *   `z`: Complex scaling factor.
        *   `p_eig_comm`: MPI communicator.
    *   **Returns**: 0 on success, non-zero on error.
*   `int p_lin(CPX *Afull, CPX *RHSfull, CPX *SOLfull, int nvec, int nrhs, MPI_Comm p_eig_comm)`:
    *   **Description**: Solves a dense complex linear system `Afull * SOLfull = RHSfull` for `SOLfull` using LU factorization.
    *   **Parameters**:
        *   `Afull`: (Input) Coefficient matrix.
        *   `RHSfull`: (Input) Right-hand side matrix/vector(s).
        *   `SOLfull`: (Output) Solution matrix/vector(s).
        *   `nvec`: Dimension of `Afull`.
        *   `nrhs`: Number of right-hand sides.
        *   `p_eig_comm`: MPI communicator.
    *   **Returns**: 0 on success, non-zero on error. Uses ScaLAPACK `pzgetrf` and `pzgetrs`.
*   `int p_lin(CPX *Afull, CPX *RHSfull, int nvec, int nrhs, CPX z, MPI_Comm p_eig_comm)`:
    *   **Description**: Solves linear systems involving `Afull` and its transpose with `RHSfull`, then combines results into `Afull` and `RHSfull`. The exact operations `Afull = RHSloc_solved_N - RSSloc_solved_T` and `RHSfull = z*RHSloc_solved_N - (1/z)*RSSloc_solved_T` need context for full interpretation.
    *   **Parameters**:
        *   `Afull`: (Input/Output) Coefficient matrix and result accumulator.
        *   `RHSfull`: (Input/Output) Right-hand side and result accumulator.
        *   `nvec`: Dimension of `Afull`.
        *   `nrhs`: Number of right-hand sides.
        *   `z`: Complex scaling factor.
        *   `p_eig_comm`: MPI communicator.
    *   **Returns**: 0 on success, non-zero on error.

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for parallel eigenvalue solution
// MPI_Comm communicator = MPI_COMM_WORLD;
// int matrix_size = 100;
// double eigenvalues[matrix_size];
// CPX* H_matrix_full; // Assume allocated and initialized on root or all processes
// CPX* S_matrix_full; // Assume allocated and initialized

// // (Potentially distribute H_matrix_full and S_matrix_full appropriately if not already)
// int error_code = p_eig(H_matrix_full, S_matrix_full, eigenvalues, matrix_size, communicator);
// if (error_code == 0) {
//     // Eigenvalues are in 'eigenvalues', eigenvectors overwrite H_matrix_full
// }
```

## Dependencies and Interactions

### Included Files
*   `#include "ParallelEig.H"`

### External Libraries
*   **ScaLAPACK**: This is the core library used for all distributed dense linear algebra operations. Specific routines include `pdsygvx` (real generalized eigensolver), `pzhegvx` (complex Hermitian generalized eigensolver), `pzgetrf` (complex LU factorization), `pzgetri` (complex matrix inverse from LU), `pzgetrs` (complex solve from LU), `pdgeadd`/`pzgeadd` (matrix redistribution/addition), and `pdlamch` (machine constants).
*   **BLACS**: Used by ScaLAPACK for process grid management and communication (e.g., `Cblacs_gridinit`, `Cblacs_gridinfo`, `Cblacs_gridexit`, `c_numroc`, `c_descinit`).
*   **MPI**: Required by BLACS and ScaLAPACK for underlying parallel communication.

### Interactions with Other Project Components
*   This file implements the functions declared in `ParallelEig.H`.
*   It provides high-level parallel linear algebra routines that can be called by other modules within OMEN that deal with dense matrices in a distributed environment, for instance, in certain types of eigenvalue problems or direct solution methods.
*   It depends on `Types.H` (via `ParallelEig.H`) for the `CPX` complex data type and potentially `LOGCERR`.
*   It uses ScaLAPACK wrapper functions (e.g., `c_pdsygvx`) which are likely declared in `ScaLapack.H`.

---
*This documentation is auto-generated and may require further manual refinement.*
