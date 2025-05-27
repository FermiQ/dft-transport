# Documentation for `src/c_scf.C`

## Overview

This file implements the `c_scf_method` function, which is a key entry point for OMEN's self-consistent field (SCF) calculations and quantum transport simulations, especially when interfacing with the CP2K quantum chemistry package. The main responsibility of `c_scf_method` is to take overlap (S) and Kohn-Sham (KS) matrices (typically from CP2K), along with various transport parameters, and compute the density matrix (P). It orchestrates the calculation flow, which can involve energy integration via `Energyvector` or a full self-consistent loop via `semiselfconsistent` (if OMEN's internal Poisson solver is used).

The file also contains several utility functions for reading, writing, and manipulating sparse matrices in the `cp2k_csr_interop_type` format used for CP2K interoperability.

## Key Components

### Functions
*   `void write_cp2k_csr(cp2k_csr_interop_type& cp2kCSRmat, const char* filename)`:
    *   **Description**: Writes the content of a `cp2k_csr_interop_type` sparse matrix to a text file.
*   `void read_nzvals_bin(cp2k_csr_interop_type& cp2kCSRmat, const char* filename, MPI_Comm cp2k_comm)`:
    *   **Description**: Reads the non-zero values of a `cp2k_csr_interop_type` sparse matrix from a binary file in parallel using MPI.
*   `void add_full_to_scaled_cp2k_csr(cp2k_csr_interop_type& cp2kCSRmat, double* Pf, int start_i_to, int start_j_to, int length_i, int length_j, double a, double b)`:
    *   **Description**: Adds a scaled dense matrix `Pf` to a sub-block of a `cp2k_csr_interop_type` sparse matrix. `cp2kCSRmat = a * Pf + b * cp2kCSRmat`.
*   `void add_from_to_scaled_cp2k_csr(cp2k_csr_interop_type& cp2kCSRmat, double a, double b, int start_i_fr, int start_j_fr, int start_i_to, int start_j_to, int length_i, int length_j, MPI_Comm cp2k_comm)`:
    *   **Description**: Extracts a sub-block from `cp2kCSRmat`, converts it to dense, and then adds it (scaled) to another (or the same) sub-block of `cp2kCSRmat`.
*   `void write_scaled_cp2k_csr_bin(cp2k_csr_interop_type& cp2kCSRmat, const char* filename, double factor, MPI_Comm cp2k_comm)`:
    *   **Description**: Writes a `cp2k_csr_interop_type` matrix to a binary file in a specific format, scaling its non-zero values by `factor`.
*   `void write_scaled_cp2k_csr_bin_remove_pbc(cp2k_csr_interop_type& cp2kCSRmat, const char* filename, double factor, int bw, int ndof, MPI_Comm cp2k_comm)`:
    *   **Description**: Writes a `cp2k_csr_interop_type` matrix to a binary file, similar to `write_scaled_cp2k_csr_bin`, but with logic to remove elements corresponding to periodic boundary condition interactions beyond a certain bandwidth `bw`.
*   `void c_scf_method(cp2k_transport_parameters cp2k_transport_params, cp2k_csr_interop_type S, cp2k_csr_interop_type KS, cp2k_csr_interop_type * P, cp2k_csr_interop_type * PImag)`:
    *   **Description**: The main function that drives the SCF or transport calculation. It converts CP2K parameters to OMEN's internal `transport_parameters`, sets up contact information, and then calls either `Energyvector::Execute` for non-SCF calculations (like transmission) or `semiselfconsistent` for full SCF loops with OMEN's Poisson solver. It also handles density matrix mixing and outputting Mulliken charges.
    *   The Doxygen comment for this function is: `/*! \brief Takes the overlap (S) and Kohn-Sham (KS) matrices as input and returns a density matrix (P). This function acts as the gate to the CP2K's world. */`

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual usage of c_scf_method
// cp2k_transport_parameters cp2k_params;
// cp2k_csr_interop_type S_matrix, KS_matrix, P_matrix, PImag_matrix;

// // ... Initialize cp2k_params and matrices S, KS, P, PImag ...
// // (P and PImag are typically allocated and have their structure defined,
// //  but their nzvals might be modified by c_scf_method)

// c_scf_method(cp2k_params, S_matrix, KS_matrix, &P_matrix, &PImag_matrix);
// // P_matrix and PImag_matrix now contain the calculated density matrix components.
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "SemiSelfConsistent.H"` (Conditionally included if `HAVE_OMEN_POISSON` is defined)
*   `#include "EnergyVector.H"`
*   `#include <numeric>` (For `std::accumulate`)

### External Libraries
*   **MPI**: Used for parallel I/O operations and within the core calculation routines.

### Interactions with Other Project Components
*   **CP2K Data Structures**: Works directly with `cp2k_csr_interop_type` and `cp2k_transport_parameters` which are data structures designed for interoperability with CP2K.
*   **`CSR.H`**: Uses `TCSR<double>` for intermediate sparse matrix manipulations.
*   **`Types.H`**: Uses `transport_parameters` and `contact_type` which are defined in `Types.H`.
*   **`EnergyVector.H`**: Calls `Energyvector::Execute` to perform energy integration for calculating density or transmission.
*   **`SemiSelfConsistent.H`**: (If `HAVE_OMEN_POISSON` is defined) Calls `semiselfconsistent` to perform a full SCF loop with OMEN's internal Poisson solver.
*   **`Blas.H`**: Uses `c_dscal` and `c_dcopy` (from `Blas.H`) for basic vector operations.

---
*This documentation is auto-generated and may require further manual refinement.*
