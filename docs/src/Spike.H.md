# Documentation for `src/Spike.H`

## Overview

This header file defines the `Spike<T>` class template, a core component of the OMEN project designed to solve large, often sparse, linear systems of equations. It implements the SPIKE algorithm, which is suitable for parallel execution on distributed memory systems using MPI. The class is templated to support different numerical types (e.g., `CPX` for complex numbers, `double` for real numbers). The algorithm involves partitioning the matrix, performing local factorizations (on `D` blocks), calculating "spike" matrices (`V`, `W`), forming a reduced system (`Sr`, `Gr`), solving it, and then reconstructing the full solution.

## Key Components

### Classes/Structs
*   `class Spike<T>`:
    *   **Description**: A template class that implements the SPIKE algorithm for solving linear systems of the form `M*X = F`.
    *   **Template Parameter**: `T` (typically `CPX` or `double`).
    *   **Public Interface**:
        *   `Spike(TCSR<T>* matrix, T* RHS, int RHS_col, MPI_Comm communicator, bool diagonal_dense = false)`: Constructor. Initializes the solver with the matrix `M` (in CSR format), the right-hand side `RHS`, number of columns in RHS, MPI communicator, and a flag for whether diagonal blocks `D` are dense.
        *   `~Spike()`: Destructor. Cleans up allocated resources like `ipiv`.
        *   `void solve_full(T* sol)`: The main public method to trigger the solution process. The solution is stored in `sol`.
        *   `void debug()`: (Conditional, only if `SPIKE_DEBUG` is defined) A debugging function, likely to dump intermediate matrices.
    *   **Private Methods (Core Algorithm)**:
        *   `void preprocess()`: Orchestrates the first phase of the SPIKE algorithm (preparing D, calculating spikes V and W, calculating G, forming the reduced system components Sr and Gr).
        *   `void postprocess()`: Orchestrates the second phase (solving the reduced system for Xr, distributing Xr, and expanding it to the full solution X).
        *   `void prepare_D()`: Extracts and factorizes the diagonal block `D` of the local partition of `M`.
        *   `void calculate_spikes()`: Computes the `V` and `W` spike matrices by solving systems involving `D` and off-diagonal blocks of `M`.
        *   `void calculate_G()`: Computes the modified right-hand side `G` by solving `D*G = F`.
        *   `void delete_D()`: Frees memory associated with the `D` block after it's used.
        *   `void prepare_send_buffer_spikes()`: Prepares buffers to send parts of `V` and `W` (the "reduced spikes") to the master rank.
        *   `void distribute_reduced_spikes()`: Gathers the reduced spike components onto the master rank to form `Sr`.
        *   `void prepare_send_buffer_G()`: Prepares buffers to send parts of `G` (the "reduced G") to the master rank.
        *   `void distribute_reduced_G()`: Gathers the reduced G components onto the master rank to form `Gr`.
        *   `void prepare_sparsity_pattern()`: (Master rank only) Prepares the sparsity pattern for the reduced system matrix `Sr`.
        *   `void solve_reduced_system()`: (Master rank only) Solves the reduced system `Sr*Xr = Gr`.
        *   `void distribute_Xr()`: Sends relevant parts of the solution `Xr` from the master rank back to other ranks.
        *   `void expand_reduced_system()`: Reconstructs the local part of the full solution `X` using `G`, `V`, `W`, and the received parts of `Xr`.
    *   **Private Methods (Utilities & Linear Algebra)**:
        *   `get_dense_from_sparse_c()`, `get_dense_from_sparse_f()`: Extract sub-blocks from sparse CSR matrices into dense C-style or Fortran-style arrays.
        *   `get_sparse_from_sparse()`: Extracts a sub-block from a sparse CSR matrix into another sparse CSR matrix.
        *   `get_max_column()`: Finds the maximum column index in a CSR matrix.
        *   `has_right_hand_side()`: Checks if an RHS `F` was provided.
        *   `set_mpi_dataype()`: Sets the `MPI_data_type` member based on `T`. (Implemented in `Spike.C`)
        *   `calculate_lu_decomposition()`: Performs LU decomposition of a dense matrix. (Implemented in `Spike.C`)
        *   `solve_linear_system_dense()`: Solves a dense linear system using a pre-computed LU decomposition. (Implemented in `Spike.C`)
        *   `solve_linear_system_sparse()`: Solves a sparse linear system (e.g., using Pardiso).
        *   `spy()`: Visualizes matrix sparsity. (Implemented in `Spike.C`)
        *   `full()`: Prints full matrix content. (Implemented in `Spike.C`)
        *   `print_array()`: Prints a 1D array. (Implemented in `Spike.C`)
        *   `merge_matrix_f()`, `merge_matrix_c()`: Merges a smaller matrix into a larger one.
        *   `get_sparse_matrix_value()`: Retrieves a single value from a sparse matrix.
        *   `transpose_dense_matrix()`: Transposes a dense matrix (out-of-place).
        *   `trans()`: Transposes a dense matrix (helper, possibly in-place or specific layout).
        *   `xLACPY()`: Wrapper for BLAS LACPY. (Implemented in `Spike.C`)
        *   `xGEMM()`: Wrapper for BLAS GEMM. (Implemented in `Spike.C`)
        *   `dump()`, `dump_c()`, `dump_f()`: (Conditional, if `SPIKE_DEBUG` is defined) Debugging functions to write matrices to files.
    *   **Private Member Variables**:
        *   `M`: Pointer to the local part of the input matrix (CSR format).
        *   `F`: Pointer to the local part of the right-hand side.
        *   `D_dense`, `D_sparse`: Diagonal block of `M` (either dense or sparse).
        *   `G`: Local part of the intermediate RHS (`D^-1 * F`).
        *   `V`, `W`: Spike matrices.
        *   `Sr`: Reduced system matrix (on master rank).
        *   `Gr`: Reduced system RHS (on master rank).
        *   `Xr`: Solution of the reduced system (on master rank).
        *   `Xr_local`: Relevant parts of `Xr` on each rank.
        *   `Sr_sendbuffer`, `Gr_sendbuffer`: Buffers for MPI communication.
        *   `X`: Pointer to where the local part of the final solution should be stored.
        *   `ipiv`: Pivot vector for LU factorization.
        *   MPI related members: `comm`, `MPI_data_type`, `win_Sr`, `win_Gr`, `win_Xr`, `win_X`, `rank`, `num_ranks`, `master_rank`, `last_rank`.
        *   Sizes, dimensions, and bandwidth information for various matrices.
        *   `debug_path`: Path for debug output files.
        *   `start_time`: For timing (if `SPIKE_TIMING` is defined).

### Macros
*   `SPIKE_DEBUG`: If defined, enables debugging code sections (e.g., `dump` functions, extra print statements).
*   `SPIKE_DEBUG_PATH`: Defines the path for dumping debug files.
*   `SPIKE_TIMING`: If defined, enables timing code sections.

## Important Variables/Constants
*No global variables or constants are defined in this header file. All variables are members of the `Spike<T>` class or local to its methods.*

## Usage Examples

```c++
// Conceptual example of using the Spike solver
// Assume matrix_A (TCSR<CPX>), rhs_B (CPX*), and solution_X (CPX*) are set up
// MPI_Comm current_communicator = MPI_COMM_WORLD;
// int num_rhs_columns = 1;
// bool diagonal_blocks_are_dense = false;

// Spike<CPX> spike_solver(&matrix_A, rhs_B, num_rhs_columns, current_communicator, diagonal_blocks_are_dense);
// spike_solver.solve_full(solution_X);
// // solution_X now contains the result
```

## Dependencies and Interactions

### Included Files
*   `#include <vector>`
*   `#include <assert.h>`
*   `#include <string.h>`
*   `#include <stdlib.h>`
*   `#include <sstream>`
*   `#include <algorithm>`
*   `#include <mpi.h>`
*   `#include "Blas.H"`
*   `#include "CSR.H"`
*   `#include "Types.H"`
*   `#include "Utilities.H"`
*   `#include "Pardiso.H"`
*   `#include "LinearSolver.H"`
*   If `SPIKE_DEBUG` is defined:
    *   `#include <cstdio>`
    *   `#include <iostream>`


### External Libraries
*   **MPI**: Essential for parallel communication and distributed memory operations.
*   **BLAS/LAPACK**: Used for dense linear algebra operations (matrix multiplication, LU factorization, solve). Indicated by `Blas.H` and functions like `xGEMM`, `xLACPY`, `calculate_lu_decomposition`.
*   **Pardiso**: Used as one of the sparse direct solvers for `solve_linear_system_sparse`. Indicated by `Pardiso.H`.

### Interactions with Other Project Components
*   **`CSR.H` (`TCSR<T>`)**: Uses the `TCSR<T>` sparse matrix format extensively.
*   **`Types.H` (`CPX`)**: Uses complex data type `CPX` and other custom types.
*   **`Utilities.H`**: Likely uses utility functions for timing, memory management, or I/O.
*   **`Blas.H`**: Uses C wrappers for BLAS/LAPACK routines.
*   **`Pardiso.H`, `LinearSolver.H`**: Interacts with other linear solver interfaces/implementations, particularly Pardiso for sparse solves.
*   The actual templated implementations of many member functions are expected to be in `Spike.C`.

---
*This documentation is auto-generated and may require further manual refinement.*
