# Documentation for `src/MUMPS.H`

## Overview

This header file defines the `MUMPS<T>` class. This class serves as an interface to the MUMPS (MUltifrontal Massively Parallel sparse direct Solver) library. MUMPS is a software package for solving large sparse systems of linear algebraic equations, particularly on distributed memory parallel computers. The `MUMPS<T>` class is a template class that inherits from OMEN's `LinearSolver<T>` interface, enabling the use of MUMPS as a backend for linear solves within the OMEN project.

## Key Components

### Functions
*   `extern "C" void MUMPS_CALL zmumps_c(ZMUMPS_STRUC_C * zmumps_par);`: Declaration of the C interface function for the MUMPS solver.

### Classes/Structs
*   `template <class T> class MUMPS : public LinearSolver<T>`:
    *   **Description**: An adapter class that wraps the MUMPS library to conform to the `LinearSolver` interface.
    *   **Template Parameter**: `T` (specialized for `CPX` - complex double-precision in the provided implementation details).
    *   **Public Members (Functions)**:
        *   `MUMPS(TCSR<T>* mat, MPI_Comm solver_comm)`: Constructor. Initializes MUMPS, sets up the problem structure by converting the input `TCSR` matrix to MUMPS's distributed input format.
        *   `MUMPS(TCSR<T>* mat, MPI_Comm solver_comm, int pord)`: Overloaded constructor. Allows specifying a MUMPS ordering strategy (`pord`).
        *   `virtual ~MUMPS()`: Destructor. Calls MUMPS to finalize and free allocated memory. (Specialized for `CPX`).
        *   `virtual void prepare()`: Prepares the solver by calling the analysis and factorization phases of MUMPS (`id.job = 4`). (Specialized for `CPX`).
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method from `LinearSolver`. Calls the main `prepare()`.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs)`: Solves the linear system `A*res = rhs` using the pre-factored matrix from `prepare()` by calling the solve phase of MUMPS (`id.job = 3`). Handles distribution of RHS and solution. (Specialized for `CPX`).
    *   **Private Members (Variables)**:
        *   `int mpi_size, mpi_rank`: MPI communicator size and rank.
        *   `int fortran_index`: Flag for Fortran-style (1-based) indexing.
        *   `int m_loc`: Number of local rows for the current MPI process.
        *   `int fst_row`: Global index of the first row on the current MPI process.
        *   `MPI_Comm slu_comm`: MPI communicator used by MUMPS.
        *   `ZMUMPS_STRUC_C id`: The main MUMPS control and data structure.
    *   **Private Members (Functions)**:
        *   `void create_local_matrix(TCSR<T>* matrix, ZMUMPS_STRUC_C* ZStruc)`: Converts the local part of the OMEN `TCSR` matrix into the format required by MUMPS (populating `ZStruc->irn_loc`, `ZStruc->jcn_loc`, `ZStruc->a_loc`). (Specialized for `CPX`).

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for MUMPS solver
// TCSR<CPX> A_sparse_matrix; // Assuming this is initialized
// CPX* b_rhs_distributed;   // Assuming this is appropriately distributed
// CPX* x_solution_distributed; // Buffer for the solution
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD;

// // ... Initialize A_sparse_matrix, b_rhs_distributed, x_solution_distributed ...

// MUMPS<CPX> solver(&A_sparse_matrix, comm);
// solver.prepare(); // Analyzes and factorizes the matrix
// solver.solve_equation(x_solution_distributed, b_rhs_distributed, num_rhs);
// // x_solution_distributed now contains the result in distributed form
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "Types.H"`
*   `#include "LinearSolver.H"`
*   `#include "zmumps_c.h"` (MUMPS C interface header)

### External Libraries
*   **MUMPS**: The primary dependency, providing distributed sparse direct solver capabilities.
*   **MPI**: Required by MUMPS for its parallel operations.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `MUMPS<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`CSR.H` (`TCSR<T>`)**: Takes a `TCSR<T>` matrix as input, which is then converted into the distributed format required by MUMPS.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX`.
*   The file primarily provides specializations for the `CPX` data type.

---
*This documentation is auto-generated and may require further manual refinement.*
