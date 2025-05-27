# Documentation for `src/Hypre.H`

## Overview

This header file defines the `Hypre<T>` class, which serves as an interface to the Hypre library. Hypre is a software library containing a collection of high-performance preconditioners and solvers for large, sparse linear systems of equations, particularly designed for parallel computing platforms. The `Hypre<T>` class allows OMEN to leverage Hypre's capabilities as one of its linear solver backends, by conforming to the `LinearSolver<T>` interface.

The file also defines a custom exception structure, `H_Exception`, for reporting errors from Hypre library calls.

## Key Components

### Structs
*   `struct H_Exception`:
    *   **Description**: A custom exception structure used to report errors originating from Hypre library functions. It captures the line number, file name, MPI rank, and Hypre error code, and can provide a textual description of the error.

### Classes
*   `template <class T> class Hypre : public LinearSolver<T>`:
    *   **Description**: An adapter class that wraps the Hypre library's solvers to fit within OMEN's `LinearSolver` framework.
    *   **Template Parameter**: `T` (data type of matrix elements, e.g., `double`, `CPX`).
    *   **Public Members (Functions)**:
        *   `Hypre(TCSR<T>* mat, MPI_Comm solver_comm)`: Constructor. Stores the input matrix (`mat`) and MPI communicator (`solver_comm`).
        *   `virtual ~Hypre()`: Destructor.
        *   `virtual void prepare()`: Implements the `LinearSolver` interface method for preparing the solver (e.g., setting up preconditioners, performing matrix factorization if applicable). Currently an empty placeholder.
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs)`: Implements the core solve functionality. It sets up Hypre's internal matrix (`HYPRE_IJMatrix`) and vector (`HYPRE_IJVector`) structures from the input `TCSR` matrix `P` and right-hand side `rhs`. It then uses a Hypre solver (specifically ParCSRBiCGSTAB with Euclid preconditioning in the example) to solve the system.
    *   **Private Members (Variables)**:
        *   `TCSR<T>* P`: Pointer to the input sparse matrix in `TCSR` format.
        *   `MPI_Comm internal_comm`: MPI communicator used for Hypre operations.
    *   **Private Members (Functions)**:
        *   `inline MPI_Datatype give_me_MPI_datatype (double*)`: Helper to return `MPI_DOUBLE`.
        *   `inline MPI_Datatype give_me_MPI_datatype (CPX*)`: Helper to return `MPI_DOUBLE_COMPLEX`.


## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for Hypre solver
// TCSR<CPX> A_matrix; // Assuming this is initialized
// CPX* b_rhs;         // Assuming this is initialized and distributed correctly
// CPX* x_solution;    // Buffer for the solution
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD;

// // ... Initialize A_matrix, b_rhs, x_solution ...

// Hypre<CPX> solver(&A_matrix, comm);
// // solver.prepare(); // If any specific Hypre setup is needed (currently empty)
// solver.solve_equation(x_solution, b_rhs, num_rhs);
// // x_solution now contains the result in distributed form
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "Types.H"`
*   `#include "LinearSolver.H"`
*   `#include "HYPRE_krylov.h"` (Hypre Krylov solvers)
*   `#include "HYPRE.h"` (Main Hypre header)
*   `#include "HYPRE_parcsr_ls.h"` (Hypre Parallel CSR Linear System interface)

### External Libraries
*   **Hypre**: The primary dependency, providing distributed sparse linear solvers and preconditioners.
*   **MPI**: Required by Hypre for its parallel operations.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `Hypre<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`CSR.H` (`TCSR<T>`)**: Takes a `TCSR<T>` matrix as input and converts its data to Hypre's distributed matrix format (`HYPRE_IJMatrix`).
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX`.
*   This class allows OMEN to use Hypre's iterative solvers (like BiCGSTAB with Euclid preconditioning) for solving large sparse linear systems that arise in its simulations.

---
*This documentation is auto-generated and may require further manual refinement.*
