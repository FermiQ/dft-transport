# Documentation for `src/SpikeSolver.H`

## Overview

This header file defines the `SpikeSolver<T>` class. It is a template class that inherits from the `LinearSolver<T>` interface. Its primary role is to adapt the SPIKE algorithm implementation (provided by the `Spike<T>` class from `Spike.H`) to fit into the common linear solver framework used in the OMEN project. This allows different solver algorithms to be used interchangeably.

## Key Components

### Classes/Structs
*   `class SpikeSolver<T> : public LinearSolver<T>`:
    *   **Description**: A template class that wraps the `Spike<T>` solver, making it usable as a `LinearSolver`.
    *   **Template Parameter**: `T` (e.g., `CPX` for complex, `double` for real).
    *   **Public Members**:
        *   `SpikeSolver(TCSR<T>* matrix, MPI_Comm communicator)`: Constructor. Takes the matrix to be solved (in `TCSR` format) and an MPI communicator.
        *   `~SpikeSolver()`: Destructor (empty, as `Spike` object is created on the stack in `solve_equation`).
        *   `void prepare()`: Implements the `LinearSolver` interface method. Currently an empty placeholder.
        *   `void prepare(int*, int*, int, int, int*, int)`: Overloaded `prepare` method from `LinearSolver` interface. Currently an empty placeholder.
        *   `void solve_equation(T* res, T* rhs, int no_rhs)`: Implements the core solve functionality. It creates an instance of `Spike<T>` and calls its `solve_full` method.
    *   **Private Members**:
        *   `TCSR<T>* _matrix`: Pointer to the sparse matrix in CSR format.
        *   `MPI_Comm _communicator`: The MPI communicator for distributed operations.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example of using SpikeSolver
// TCSR<CPX> A_matrix;
// CPX* b_rhs;
// CPX* x_solution;
// int num_rhs = 1;
// MPI_Comm comm = MPI_COMM_WORLD;

// // ... Initialize A_matrix, b_rhs, x_solution ...

// SpikeSolver<CPX> solver(&A_matrix, comm);
// // solver.prepare(); // If needed, though current implementation is empty
// solver.solve_equation(x_solution, b_rhs, num_rhs);
// // x_solution now contains the result
```

## Dependencies and Interactions

### Included Files
*   `#include <vector>`
*   `#include <mpi.h>`
*   `#include "LinearSolver.H"`
*   `#include "Spike.H"`
*   `#include <omp.h>`

### External Libraries
*   **MPI**: Used for parallel processing, indicated by `<mpi.h>` and `MPI_Comm`.
*   **OpenMP**: Indicated by `<omp.h>` and a debug print statement in the constructor about thread count.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `SpikeSolver<T>` inherits from the `LinearSolver<T>` abstract class/interface.
*   **`Spike.H` (`Spike<T>`)**: `SpikeSolver<T>` uses an instance of `Spike<T>` to perform the actual linear system solution.
*   **`CSR.H` (`TCSR<T>`)**: Uses the `TCSR<T>` sparse matrix format.
*   The implementation relies on the `Spike<T>` class to handle the details of the SPIKE algorithm.

---
*This documentation is auto-generated and may require further manual refinement.*
