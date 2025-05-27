# Documentation for `src/SplitSolve.H`

## Overview

This header file defines the `SplitSolve<T>` class. This is a template class designed as a linear solver, inheriting from `LinearSolver<CPX>` (note: it inherits from `LinearSolver<CPX>` specifically, while being templated itself with `T`). The implementation suggests a sophisticated solver strategy that likely involves splitting the problem into blocks, potentially for execution on hybrid CPU/GPU systems, given the presence of CUDA, cuBLAS, and MAGMA library calls. It uses MPI for distributed memory parallelism.

The solver appears to go through multiple stages (`FirstStage` through `SixthStage`, plus a `RecursiveStage`) to compute the solution.

## Key Components

### Classes/Structs
*   `template <class T> class SplitSolve : public LinearSolver<CPX>`:
    *   **Description**: A linear solver implementation, possibly optimized for specific matrix structures or hardware configurations (e.g., GPUs).
    *   **Template Parameter**: `T` (likely `double` or `CPX`, as specializations for `extract_diag`, `change_var_type`, etc. are provided for these).
    *   **Public Members**:
        *   `SplitSolve(TCSR<CPX>* mat, double* pHostMem, int pCPU_per_bc, int pinterleave, MPI_Comm solver_comm, MPI_Comm glob_solver_comm)`: Constructor.
        *   `virtual ~SplitSolve()`: Destructor.
        *   `virtual void prepare()`: Implements the `LinearSolver` interface.
        *   `virtual void prepare(int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: An overloaded prepare method, likely to set up blocking strategies.
        *   `virtual void prepare_corner(CPX* ML_ref, CPX* MR_ref, int* NBC, int* Bmin, int* Bmax, int NBlock, int Bsize, int* orb_per_at, int tb)`: A specialized prepare method, possibly for handling boundary conditions or corner parts of the matrix.
        *   `virtual void solve_equation(CPX* res, CPX* rhs, int no_rhs)`: The main method to solve the linear system `matrix * res = rhs`.
    *   **Private Methods (subset, for brevity - many stages and helpers)**:
        *   `void extract_diag(...)`, `void extract_not_diag(...)`: GPU kernels or wrappers to extract diagonal and off-diagonal blocks.
        *   `void change_var_type(...)`: Converts data type on the GPU (e.g., double to CPX).
        *   `void symmetrize_matrix(...)`: Symmetrizes a matrix on the GPU.
        *   `void create_blocks(...)`: Defines how the matrix is partitioned into blocks.
        *   `void FirstStage()` through `void SixthStage()`: Sequential stages of the algorithm.
        *   `void RecursiveStage()`: A recursive part of the algorithm.
        *   `void InitSolve(...)`, `void FirstSolve(...)` through `void FourthSolve(...)`: Stages involved in the actual solution process using the precomputed factors/blocks.
        *   `void write_dev_matrix(...)`, `void write_matrix(...)`: Utility functions for writing matrix data (from device or host) to files, likely for debugging.
    *   **Private Member Variables (subset, for brevity)**:
        *   MPI-related: `glob_rank`, `glob_size`, `left_rank`, `right_rank`, `mpi_size`, `mpi_rank`, `wco_comm`, `glob_wco_comm`.
        *   CUDA-related: `stream_c`, `stream_m` (CUDA streams), `cublas_handle`.
        *   Host memory pointers: `HostMem`, `M_host`, `M1_host`.
        *   Device memory pointers: `M_dev`, `M1_dev`, `M2_dev`, `C_dev`, `xLU_dev`, `SigmaB_dev`, `bL_dev`, `bR_dev`, `M3_dev`, `y_dev`, `z_dev`, `nnz_dev`, `edge_i_dev`, `index_j_dev`.
        *   Problem/block definition: `matrix` (input `TCSR<CPX>*`), `n_block`, `b_size`, `IL_first`, `b_min`, `b_max`.

## Important Variables/Constants
*   *None defined globally in this file. All variables are members of the `SplitSolve<T>` class or local to its methods.*

## Usage Examples

```c++
// Conceptual example for SplitSolve
// TCSR<CPX> A_matrix;
// CPX* b_rhs;
// CPX* x_solution;
// double* host_memory_buffer; // Pre-allocated host memory
// int cpu_per_boundary_condition = 1;
// int interleave_factor = 0;
// MPI_Comm local_comm = MPI_COMM_WORLD, global_comm = MPI_COMM_WORLD;
// int num_rhs = 1;

// // ... Initialize A_matrix, b_rhs, x_solution, host_memory_buffer ...
// // ... Define Bmin, Bmax, NBlock, Bsize, orb_per_atom, tb for prepare ...

// SplitSolve<double> solver(&A_matrix, host_memory_buffer, cpu_per_boundary_condition,
//                           interleave_factor, local_comm, global_comm);
// // solver.prepare(Bmin, Bmax, NBlock, Bsize, orb_per_atom, tb); // If needed
// solver.solve_equation(x_solution, b_rhs, num_rhs);
// // x_solution now contains the result
```

## Dependencies and Interactions

### Included Files
*   `#include "cublas_v2.h"`
*   `#include "CSR.H"`
*   `#include "CSC.H"`
*   `#include "Types.H"`
*   `#include "Blas.H"`
*   `#include "LinearSolver.H"`

### External Libraries
*   **CUDA**: Indicated by CUDA streams (`cudaStream_t`), device memory allocation (`allocate_data_on_device`), and CUDA specific operations (e.g. `d_extract_diag_on_dev`).
*   **cuBLAS**: Indicated by `cublas_v2.h` and `cublasHandle_t`, used for BLAS operations on the GPU.
*   **MAGMA**: Indicated by `magma_init()`, `magma_finalize()`, and `magma_zgesv_nopiv_gpu`, suggesting use of MAGMA for GPU-accelerated linear algebra routines.
*   **MPI**: Used for distributed memory parallelism, indicated by `MPI_Comm` and MPI calls.

### Interactions with Other Project Components
*   **`LinearSolver.H`**: `SplitSolve<T>` inherits from the `LinearSolver<CPX>` interface.
*   **`CSR.H` (`TCSR<CPX>`)**: Takes a `TCSR<CPX>*` matrix as input.
*   **`CSC.H`**: Included, but direct usage of CSC structures is not immediately apparent from member variables.
*   **`Types.H` (`CPX`)**: Uses the complex data type `CPX`.
*   **`Blas.H`**: Likely uses CPU-side BLAS wrappers for some operations or type definitions compatible with BLAS.
*   The solver is highly dependent on a specific setup involving MPI ranks, CPU configurations (`CPU_per_bc`, `interleave`), and available GPU resources.

---
*This documentation is auto-generated and may require further manual refinement.*
