# Documentation for `src/SuperLU.H`

## Overview

This header file defines the `SuperLU<T>` class, which acts as an interface to the SuperLU_DIST library. SuperLU_DIST is a high-performance solver for large, sparse, non-symmetric systems of linear equations, designed for distributed memory parallel computing environments (using MPI). The `SuperLU<T>` class is a template class that inherits from OMEN's `LinearSolver<T>` interface, allowing SuperLU_DIST to be used as a backend for solving linear systems within the OMEN project.

## Key Components

### Classes/Structs
*   `template <class T> class SuperLU : public LinearSolver<T>`:
    *   **Description**: A wrapper class that adapts the SuperLU_DIST library to the `LinearSolver` interface used in OMEN.
    *   **Template Parameter**: `T` (typically `CPX` for complex numbers, as specializations are provided for `CPX`).
    *   **Public Members**:
        *   `SuperLU(TCSR<T>* mat, MPI_Comm solver_comm)`: Constructor. Initializes the SuperLU solver with the matrix to be solved (in `TCSR` format) and an MPI communicator.
        *   `virtual ~SuperLU()`: Destructor. Frees resources allocated by SuperLU_DIST. (Specialized for `CPX`).
        *   `virtual void prepare()`: Prepares the solver by performing matrix factorization using SuperLU_DIST routines (e.g., `pzgssvx` with factorization option). (Specialized for `CPX`).
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method from `LinearSolver`. Calls the main `prepare()`.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs)`: Solves the linear system `A*res = rhs` using the pre-factored matrix from `prepare()`. (Specialized for `CPX`).
    *   **Private Methods**:
        *   `void create_local_matrix(TCSR<T>* matrix, SuperMatrix* SMat)`: Converts the local part of the OMEN `TCSR` matrix into the SuperLU_DIST `SuperMatrix` format. (Specialized for `CPX`).
    *   **Private Member Variables**:
        *   `mpi_size`, `mpi_rank`: Size and rank of the MPI communicator.
        *   `fortran_index`: Fortran indexing flag from the input matrix.
        *   `m_loc`: Number of local rows in the matrix for the current MPI rank.
        *   `n`: Total number of columns/rows in the global matrix.
        *   `slu_comm`: MPI communicator used by SuperLU_DIST.
        *   `A`: `SuperMatrix` object representing the distributed matrix for SuperLU_DIST.
        *   `options`: `superlu_dist_options_t` struct holding options for the SuperLU_DIST solver.
        *   `grid`: `gridinfo_t` struct describing the process grid for SuperLU_DIST.
        *   `stat`: `SuperLUStat_t` struct for collecting statistics from SuperLU_DIST.
        *   `ScalePermstruct`: `ScalePermstruct_t` struct for scaling and permutation information.
        *   `LUstruct`: `LUstruct_t` struct storing the LU factors.
        *   `SOLVEstruct`: `SOLVEstruct_t` struct for the solve phase.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for SuperLU solver
// TCSR<CPX> A_matrix; // Assuming this is initialized
// CPX* b_rhs;         // Assuming this is initialized
// CPX* x_solution;    // Buffer for the solution
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD;

// // ... Initialize A_matrix, b_rhs, x_solution ...

// SuperLU<CPX> solver(&A_matrix, comm);
// solver.prepare(); // Factorizes the matrix A_matrix
// solver.solve_equation(x_solution, b_rhs, num_rhs);
// // x_solution now contains the result
```

## Dependencies and Interactions

### Included Files
*   `#include "superlu_zdefs.h"` (SuperLU_DIST specific definitions for complex types)
*   `#include "CSR.H"`
*   `#include "Types.H"`
*   `#include "LinearSolver.H"`

### External Libraries
*   **SuperLU_DIST**: The primary dependency. Indicated by `superlu_zdefs.h` and numerous calls to SuperLU_DIST functions (e.g., `superlu_gridinit`, `pzgssvx`, `Destroy_CompRowLoc_Matrix_dist`).
*   **MPI**: Required by SuperLU_DIST for distributed memory parallelism.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `SuperLU<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`CSR.H` (`TCSR<T>`)**: Uses the `TCSR<T>` sparse matrix format as input, which is then converted to SuperLU's internal format.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX`.
*   The file contains commented-out sections that seem to be for a `double` specialization, suggesting it was primarily used or completed for `CPX` (complex) matrices.

---
*This documentation is auto-generated and may require further manual refinement.*
