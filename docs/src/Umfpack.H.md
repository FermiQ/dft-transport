# Documentation for `src/Umfpack.H`

## Overview

This header file defines the `Umfpack<T>` class. This class serves as an interface or wrapper for the UMFPACK library, which is part of the SuiteSparse package. UMFPACK is a set of routines for solving unsymmetric sparse linear systems of equations, `Ax=b`. The `Umfpack<T>` class is a template class that inherits from OMEN's `LinearSolver<T>` interface, enabling UMFPACK to be used as one of the linear solver backends within the OMEN project. The note at the beginning suggests considerations for using `UF_long` for larger memory requirements.

## Key Components

### Typedefs
*   `typedef SuiteSparse_long UF_long;`: Defines `UF_long` as an alias for `SuiteSparse_long`, typically a 64-bit integer type used by UMFPACK for indexing large matrices.

### Classes/Structs
*   `template <class T> class Umfpack : public LinearSolver<T>`:
    *   **Description**: A wrapper class that adapts the UMFPACK library to the `LinearSolver` interface.
    *   **Template Parameter**: `T` (specialized for `CPX` and `double`).
    *   **Public Members**:
        *   `Umfpack(TCSR<T>* mat, MPI_Comm solver_comm)`: Constructor. Initializes UMFPACK settings, converts the input `TCSR` matrix to `TCSC` format (column-major, which UMFPACK uses), and sets up control parameters. `solver_comm` is taken but UMFPACK is primarily a serial solver.
        *   `virtual ~Umfpack()`: Destructor. Frees memory allocated by UMFPACK for symbolic and numeric factorizations. (Specialized for `CPX` and `double`).
        *   `virtual void prepare()`: Prepares the solver by performing symbolic and numeric factorization of the matrix using UMFPACK routines (e.g., `umfpack_zl_symbolic`, `umfpack_zl_numeric`). (Specialized for `CPX` and `double`).
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method from `LinearSolver`. Calls the main `prepare()`.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method from `LinearSolver`. Currently an empty placeholder.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs)`: Solves the linear system `Ax = rhs` using the pre-factored matrix from `prepare()` via UMFPACK routines (e.g., `umfpack_zl_wsolve`). (Specialized for `CPX` and `double`).
    *   **Private Member Variables**:
        *   `matrix`: (Type `TCSC<T,UF_long>*`) Pointer to the matrix in CSC (Compressed Sparse Column) format, which is suitable for UMFPACK.
        *   `n`: (Type `int`) Dimension of the matrix.
        *   `Wi`: (Type `UF_long*`) Integer work array for UMFPACK.
        *   `W`: (Type `double*`) Double work array for UMFPACK.
        *   `factorized`: (Type `bool`) Flag indicating whether the matrix has been factorized.
        *   `Info`: (Type `double[UMFPACK_INFO]`) Array to store information output from UMFPACK routines.
        *   `Control`: (Type `double[UMFPACK_CONTROL]`) Array to control various UMFPACK parameters.
        *   `Symbolic`: (Type `void*`) Pointer to the symbolic factorization object from UMFPACK.
        *   `Numeric`: (Type `void*`) Pointer to the numeric factorization object from UMFPACK.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for Umfpack solver
// TCSR<CPX> A_csr_matrix; // Assuming this is initialized
// CPX* b_rhs;            // Assuming this is initialized
// CPX* x_solution;       // Buffer for the solution
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD; // Though UMFPACK is serial

// // ... Initialize A_csr_matrix, b_rhs, x_solution ...

// Umfpack<CPX> solver(&A_csr_matrix, comm);
// solver.prepare(); // Factorizes the matrix
// solver.solve_equation(x_solution, b_rhs, num_rhs);
// // x_solution now contains the result
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "CSC.H"`
*   `#include "Types.H"`
*   `#include "LinearSolver.H"`
*   `#include "umfpack.h"`
*   `#include "amd.h"`

### External Libraries
*   **UMFPACK**: The primary dependency, part of SuiteSparse. Indicated by `umfpack.h` and calls to `umfpack_*` routines.
*   **AMD**: (Approximate Minimum Degree ordering library), also part of SuiteSparse, often used by UMFPACK. Indicated by `amd.h`.
*   **MPI**: An `MPI_Comm` is passed to the constructor, but UMFPACK itself is a serial solver. This might be for compatibility with the `LinearSolver` interface or for use in contexts where other parts of the system use MPI.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `Umfpack<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`CSR.H` (`TCSR<T>`)**: Takes a `TCSR<T>` matrix as input.
*   **`CSC.H` (`TCSC<T, UF_long>`)**: Converts the input `TCSR<T>` matrix to `TCSC<T, UF_long>` format internally for use with UMFPACK.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX` and other types.

---
*This documentation is auto-generated and may require further manual refinement.*
