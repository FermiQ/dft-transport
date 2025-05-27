# Documentation for `src/Banded.H`

## Overview

This header file defines the `Banded<T>` class. This class is designed as an interface for solving linear systems where the coefficient matrix is banded. It is a template class that inherits from the `LinearSolver<T>` interface, allowing it to be used as a specific solver backend within the OMEN project. The implementation details suggest it uses ScaLAPACK routines for distributed memory parallel processing, making it suitable for large-scale banded systems.

The Doxygen comment `/*! \class Banded \brief Interface for banded matrix solver */` confirms its purpose.

## Key Components

### Classes/Structs
*   `template <class T> class Banded : public LinearSolver<T>`:
    *   **Description**: A linear solver specifically for banded matrices, likely using ScaLAPACK.
    *   **Template Parameter**: `T` (specialized for `CPX` - complex double-precision).
    *   **Public Members**:
        *   `Banded(TCSR<T>* mat, MPI_Comm solver_comm)`: Constructor. Initializes the solver with the matrix in `TCSR` format and an MPI communicator. Default pivoting is likely used.
        *   `Banded(TCSR<T>* mat, MPI_Comm solver_comm, int pp)`: Constructor. `pp` likely controls pivoting strategy (e.g., 0 for no pivoting, 1 for partial pivoting). *Note: The implementation shown only has one constructor specialized for `CPX` which sets `pp=1` internally.*
        *   `virtual ~Banded()`: Destructor. Frees allocated memory and exits the BLACS grid. (Specialized for `CPX`).
        *   `virtual void prepare()`: Prepares the solver by performing the factorization of the banded matrix using ScaLAPACK routines (e.g., `pzgbtrf` or `pzdbtrf`). (Specialized for `CPX`).
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method from `LinearSolver`. Calls the main `prepare()`.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs)`: Solves the linear system `A*res = rhs` using the pre-factored banded matrix and ScaLAPACK routines (e.g., `pzgbtrs` or `pzdbtrs`). (Specialized for `CPX`).
    *   **Private Member Variables**:
        *   `icontxt`: (Type `int`) BLACS context handle.
        *   `pp`: (Type `int`) Pivoting flag (0 or 1).
        *   `bw`: (Type `int`) Bandwidth of the matrix.
        *   `NB`: (Type `int`) ScaLAPACK block size.
        *   `size_tot`: (Type `int`) Total size of the global matrix.
        *   `size_csr_loc`: (Type `int`) Local size of the matrix on the current MPI rank (number of rows).
        *   `LAF`: (Type `int`) Size of the `AF` array (factored matrix).
        *   `descA[7]`: (Type `int[]`) Array descriptor for the distributed matrix `A` in ScaLAPACK.
        *   `ipiv`: (Type `int*`) Pointer to the pivot array, used if `pp` is enabled.
        *   `A`: (Type `CPX*`) Pointer to the local part of the banded matrix data, stored in ScaLAPACK's banded format.
        *   `AF`: (Type `CPX*`) Pointer to the factored matrix data.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for Banded solver
// TCSR<CPX> A_banded_matrix; // Assuming this is initialized and is banded
// CPX* b_rhs;               // Assuming this is initialized
// CPX* x_solution;          // Buffer for the solution
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD;

// // ... Initialize A_banded_matrix, b_rhs, x_solution ...

// Banded<CPX> solver(&A_banded_matrix, comm);
// solver.prepare(); // Factorizes the matrix
// solver.solve_equation(x_solution, b_rhs, num_rhs);
// // x_solution now contains the result
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "Types.H"`
*   `#include "LinearSolver.H"`

### External Libraries
*   **ScaLAPACK**: The primary dependency for distributed banded matrix operations. Implied by functions like `c_pzgbtrf`, `c_pzdbtrf`, `c_pzgbtrs`, `c_pzdbtrs`, and the use of array descriptors (`descA`).
*   **BLACS**: Used by ScaLAPACK for process grid management and communication (e.g., `Cblacs_gridinit`, `Cblacs_gridexit`).
*   **MPI**: Required by BLACS and ScaLAPACK for distributed memory parallelism.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `Banded<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`CSR.H` (`TCSR<T>`)**: Takes a `TCSR<T>` matrix as input and converts it to ScaLAPACK's banded storage format via `mat->sparse_to_narrow_band()`.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX`.
*   The file primarily provides specializations for `CPX` data type.

---
*This documentation is auto-generated and may require further manual refinement.*
