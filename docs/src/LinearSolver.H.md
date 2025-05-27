# Documentation for `src/LinearSolver.H`

## Overview

This header file defines the `LinearSolver<T>` class. This class serves as an abstract base class, establishing an interface for various linear algebraic equation solvers used within the OMEN project. By defining a common set of virtual functions, it allows different solver libraries or custom solver implementations (e.g., for sparse or dense systems, direct or iterative methods) to be used interchangeably throughout the codebase. Concrete solver classes will inherit from `LinearSolver<T>` and provide specific implementations for its pure virtual methods.

## Key Components

### Classes/Structs
*   `template <class T> class LinearSolver`:
    *   **Description**: An abstract base class for linear solvers. It defines a common interface that all concrete linear solver implementations must follow.
    *   **Template Parameter**: `T` (typically `double` or `CPX`, representing the data type of the matrix elements).
    *   **Public Members (Functions)**:
        *   `LinearSolver(TCSR<T>* mat, MPI_Comm)`: Constructor, likely taking a sparse matrix and MPI communicator.
        *   `LinearSolver(TCSR<T>* mat, MPI_Comm, int)`: Overloaded constructor.
        *   `LinearSolver(TCSR<T>* mat, int, int, MPI_Comm, MPI_Comm)`: Overloaded constructor.
        *   `LinearSolver(TCSR<T>* mat, double*, int, int, MPI_Comm, MPI_Comm)`: Overloaded constructor.
        *   `LinearSolver()`: Default constructor.
        *   `virtual ~LinearSolver()`: Virtual destructor, allowing proper cleanup of derived class resources.
        *   `virtual void prepare() = 0;`: Pure virtual function. Derived classes must implement this to perform any necessary setup or factorization before solving.
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb) = 0;`: Pure virtual function. An overloaded version of `prepare`, possibly for solvers that operate on block matrices or require more detailed setup information.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb) = 0;`: Pure virtual function. A specialized preparation step, potentially for handling boundary conditions or specific matrix corner elements.
        *   `virtual void solve_equation(T* res, T* rhs, int no_rhs) = 0;`: Pure virtual function. Derived classes must implement this to solve the linear system `Ax = rhs` for `x` (stored in `res`), given one or more right-hand sides.

### Functions
*   *None defined outside the class.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// This is an abstract class and cannot be instantiated directly.
// Concrete derived classes like SuperLU, MUMPS, Full, etc., would be used:

// LinearSolver<CPX>* solver;
// if (use_superlu) {
//     solver = new SuperLU<CPX>(matrix_A, comm);
// } else if (use_mumps) {
//     solver = new MUMPS<CPX>(matrix_A, comm);
// }
// // ...
// solver->prepare();
// solver->solve_equation(solution_x, right_hand_side_b, num_rhs);
// delete solver;
```

## Dependencies and Interactions

### Included Files
*   `#include "Types.H"`
*   `#include "CSR.H"`
*   `#include "mpi.h"`

### External Libraries
*   **MPI**: Indicated by the inclusion of `mpi.h` and its use in constructor signatures, suggesting that implementations are expected to be MPI-aware.

### Interactions with Other Project Components
*   This header file defines a crucial abstract interface for the OMEN project.
*   Various concrete solver classes (e.g., `Full.H`, `Banded.H`, `SuperLU.H`, `MUMPS.H`, `Hypre.H`, `Umfpack.H`, `SplitSolve.H`) inherit from `LinearSolver<T>` and provide specific implementations for solving linear systems.
*   It uses `TCSR<T>` (from `CSR.H`) for sparse matrix representation and `CPX` (from `Types.H`) for complex number representation.
*   Modules that require solving linear systems (e.g., in the `Density.C` or `GetSigma.C` calculations) will typically use a pointer or reference to `LinearSolver<T>` to interact with a specific solver instance.

---
*This documentation is auto-generated and may require further manual refinement.*
