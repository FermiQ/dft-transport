# Documentation for `src/Connection.H`

## Overview

This header file provides a collection of template functions designed to facilitate MPI (Message Passing Interface) communication within the OMEN project. It offers specialized routines for sending, receiving, and broadcasting data, particularly for custom sparse matrix formats (`TCSR<T>`) and dense arrays. The templates are explicitly specialized for `double` and `CPX` (complex double-precision) data types.

## Key Components

### Functions
*(All functions are templates, many with explicit specializations for `double` and `CPX`)*

*   `template <typename T> void MPI_Send_Sparse(TCSR<T> *M, int dest, int message, MPI_Comm comm);`
    *   **Description**: Sends a sparse matrix `M` (in `TCSR` format) to a specified MPI destination `dest`. It sends matrix dimensions, non-zero count, and the structural arrays (`edge_i`, `index_j`) and non-zero values (`nnz`).
*   `template <typename T> void MPI_Recv_Sparse(TCSR<T> *M, int source, int message, MPI_Comm comm);`
    *   **Description**: Receives a sparse matrix `M` (in `TCSR` format) from a specified MPI source. Assumes `M` is pre-allocated with correct sizes or that sizes are received first (the implementation sends size first).
*   `template <typename T> void MPI_Send_Full(T *M, int count, int dest, int message, MPI_Comm comm);`
    *   **Description**: Sends a dense array `M` of `count` elements to a specified MPI destination. The count is sent first.
*   `template <typename T> void MPI_Isend_Full(T *M, int count, int dest, int message, MPI_Comm comm);`
    *   **Description**: Sends a dense array `M` non-blockingly to a specified MPI destination. The count is sent first.
*   `template <typename T> void MPI_Recv_Full(T *M, int source, int message, MPI_Comm comm);`
    *   **Description**: Receives a dense array `M` from a specified MPI source. The number of elements to receive is first received from the sender.
*   `template <typename T> void MPI_Recv_Full(T *M, int count, int source, int message, MPI_Comm comm);`
    *   **Description**: Receives a dense array `M` of a known `count` elements from a specified MPI source. An initial integer is received (likely count from sender for verification) but `count` parameter dictates receive buffer size.
*   `template <typename T> void MPI_Dist_Sparse(TCSR<T> *M, int source, MPI_Comm comm);`
    *   **Description**: Broadcasts the structural arrays (`index_i`, `edge_i`, `index_j`, `diag_pos`) and non-zero values (`nnz`) of a sparse matrix `M` from a source rank. Assumes matrix dimensions (`size`, `n_nonzeros`) are already consistent across ranks or broadcast separately.
*   `template <typename T> void MPI_Bcast_SpInd(TCSR<T> **M, int **M_index, int with_index, int trans_mpi_rank, MPI_Comm trans_comm);`
    *   **Description**: Broadcasts sparse matrix metadata (dimensions, findx) and then its structure and data. If `trans_mpi_rank` is not 0, it allocates memory for `*M` and `*M_index`. Optionally broadcasts `M_index` if `with_index` is true.
*   `void MPI_Send_Res(CPX *M, int N, int Ntot, int no_rhs, int dest, MPI_Comm comm);`
    *   **Description**: Sends result data (specifically `CPX` type), likely for multiple right-hand sides. Sends dimensions first, then data for each RHS.
*   `void MPI_Recv_Res(CPX *M, int source, MPI_Comm comm);`
    *   **Description**: Receives result data sent by `MPI_Send_Res`.
*   `void MPI_Bcast_Sparse(TCSR<CPX> *M, int source, MPI_Comm comm);`
    *   **Description**: Broadcasts all components of a `TCSR<CPX>` matrix (dimensions, structure, and non-zero values) from a source rank.

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined in this file.*

## Usage Examples

```c++
// Conceptual example for sending/receiving a TCSR matrix
// TCSR<CPX> matrix_to_send;
// TCSR<CPX> matrix_to_receive; // Assume allocated
// // ... populate matrix_to_send on rank 0 ...

// if (my_rank == 0) {
//     MPI_Send_Sparse(&matrix_to_send, 1, 0, MPI_COMM_WORLD);
// } else if (my_rank == 1) {
//     // Need to pre-allocate matrix_to_receive or modify MPI_Recv_Sparse
//     // to handle allocation based on received sizes.
//     // For simplicity, assume matrix_to_receive is ready for data.
//     MPI_Recv_Sparse(&matrix_to_receive, 0, 0, MPI_COMM_WORLD);
// }
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   Implicit dependencies on `CSR.H` (for `TCSR<T>`) and `Types.H` (for `CPX`) through function signatures.

### External Libraries
*   **MPI**: This entire file consists of MPI communication routines.

### Interactions with Other Project Components
*   This file provides essential MPI communication utilities for sparse matrices (`TCSR<T>`) and dense arrays, which are likely used by various modules in OMEN for parallel data exchange.
*   It depends on the definition of `TCSR<T>` (likely from `CSR.H`) and `CPX` (from `Types.H`).

---
*This documentation is auto-generated and may require further manual refinement.*
