# Documentation for `src/Full.H`

## Overview

This header file defines the `Full<T>` class. This class serves as an interface for a dense distributed linear solver within the OMEN project. It is a template class that inherits from the `LinearSolver<T>` base class. The implementation details suggest it utilizes ScaLAPACK for performing LU decomposition and solving linear systems on dense matrices distributed across multiple MPI processes.

The Doxygen comment `/*! \class Full \brief Interface for dense distributed solver */` clearly states its purpose.

## Key Components

### Classes/Structs
*   `template <class T> class Full : public LinearSolver<T>`:
    *   **Description**: An interface for a dense distributed linear solver, using ScaLAPACK.
    *   **Template Parameter**: `T` (specialized for `CPX` - complex double-precision).
    *   **Public Members (Functions)**:
        *   `Full(TCSR<T>* mat, MPI_Comm solver_comm)`: Constructor. Initializes the ScaLAPACK process grid, converts the input sparse matrix (`mat`) to a dense, distributed ScaLAPACK format (`Aloc`).
        *   `virtual ~Full()`: Destructor. Frees allocated memory and exits the BLACS grids. (Specialized for `CPX`).
        *   `virtual void prepare()`: Prepares the solver by performing the LU factorization of the distributed dense matrix `Aloc` using ScaLAPACK's `pzgetrf` routine. (Specialized for `CPX`).
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method from `LinearSolver`. Calls the main `prepare()`.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs)`: Solves the linear system `A*res = rhs` using the pre-factored matrix and ScaLAPACK's `pzgetrs` routine. It handles the distribution and gathering of the right-hand side and solution vectors. (Specialized for `CPX`).
    *   **Private Members (Variables)**:
        *   `int size_tot`: Total size (number of rows/columns) of the global matrix.
        *   `int size_csr_loc`: Local number of rows for the initial CSR matrix format.
        *   `int icontxt`: BLACS context handle for the main ScaLAPACK grid.
        *   `int icontxt_csr`: BLACS context handle for a temporary grid used during CSR to dense conversion.
        *   `int nbl`: ScaLAPACK block size for matrix distribution.
        *   `int NB_csr`: Block size used during CSR to dense conversion.
        *   `int descAloc[9]`: Array descriptor for the distributed dense matrix `Aloc` in ScaLAPACK format.
        *   `int *ipiv`: Pointer to the pivot array from LU factorization.
        *   `CPX *Aloc`: Pointer to the local part of the distributed dense matrix.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for Full solver
// TCSR<CPX> A_sparse_matrix; // Assuming this is initialized
// CPX* b_rhs_distributed;   // Assuming this is appropriately distributed or handled
// CPX* x_solution_distributed; // Buffer for the solution
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD;

// // ... Initialize A_sparse_matrix, b_rhs_distributed, x_solution_distributed ...

// Full<CPX> solver(&A_sparse_matrix, comm);
// solver.prepare(); // Factorizes the matrix
// solver.solve_equation(x_solution_distributed, b_rhs_distributed, num_rhs);
// // x_solution_distributed now contains the result in distributed form
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "Types.H"`
*   `#include "LinearSolver.H"`

### External Libraries
*   **ScaLAPACK**: The primary dependency for distributed dense linear algebra operations. Implied by function calls like `c_descinit`, `c_numroc`, `c_pzgemr2d`, `c_pzgetrf`, `c_pzgetrs`.
*   **BLACS**: Used by ScaLAPACK for process grid management and communication (e.g., `Cblacs_gridinit`, `Cblacs_gridinfo`, `Cblacs_gridexit`).
*   **MPI**: Required by BLACS and ScaLAPACK for distributed memory parallelism.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `Full<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`CSR.H` (`TCSR<T>`)**: Takes a `TCSR<T>` matrix as input, which is then converted to a dense distributed format using `mat->sparse_to_full()` and ScaLAPACK's redistribution routine `c_pzgemr2d`.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX`.
*   The file primarily provides specializations for the `CPX` data type.

---
*This documentation is auto-generated and may require further manual refinement.*
