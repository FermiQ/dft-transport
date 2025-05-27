# Documentation for `src/Density.C`

## Overview

This file implements the `density` function, which is a core computational routine in the OMEN project. Its primary responsibility is to calculate various physical quantities at a given energy point, such as the density matrix (real and imaginary parts), atom-resolved density of states (DOS), and transmission probability. The function is designed to work with different quantum transport formalisms, including Equilibrium (EQ), Non-Equilibrium Green's Function (NEGF), Green's Function (GF), and Wave Function (WF) methods. It achieves this by interfacing with a variety of linear algebra solvers and self-energy calculation modules.

The Doxygen comment `/*! \brief Function that calls the inversion or linear solvers, adds up the LDOS computed from the solution to the P matrix, and computes the atom-resolved DOS and transmission for each energy point */` summarizes its role.

## Key Components

### Functions
*   `int density(TCSR<double> *KohnSham, TCSR<double> *Overlap, TCSR<double> *Ps, TCSR<double> *PsImag, CPX energy, CPX weight, CPX dweight, transport_methods::transport_method_type method, std::vector<double> muvec, std::vector<contact_type> contactvec, std::vector<result_type> &resultvec, std::vector<int> Bsizes, std::vector<int> orb_per_at, transport_parameters transport_params, MPI_Comm matrix_comm)`:
    *   **Description**: Calculates density matrices and other transport quantities.
    *   **Parameters**:
        *   `KohnSham`: Pointer to the Kohn-Sham matrix (real part, CSR format).
        *   `Overlap`: Pointer to the Overlap matrix (CSR format).
        *   `Ps`: Pointer to the output real part of the density matrix (or related quantity, CSR format).
        *   `PsImag`: Pointer to the output imaginary part of the density matrix (CSR format).
        *   `energy`: Complex energy point (E + i*eta) at which to calculate.
        *   `weight`: Weight factor for the energy point (e.g., for integration).
        *   `dweight`: Derivative of the weight factor (used for Fermi function derivative calculations).
        *   `method`: The transport formalism to use (e.g., `transport_methods::NEGF`).
        *   `muvec`: Vector of chemical potentials for the contacts.
        *   `contactvec`: Vector of `contact_type` structs describing device contacts.
        *   `resultvec`: Vector of `result_type` structs to store calculated results like transmission and DOS.
        *   `Bsizes`: Vector of block sizes for block tridiagonal solvers (e.g., RGF).
        *   `orb_per_at`: Vector detailing orbitals per atom.
        *   `transport_params`: Structure containing various parameters for the transport calculation.
        *   `matrix_comm`: MPI communicator for matrix operations.
    *   **Returns**: `0` on success, non-zero on failure (e.g., if `LOGCERR` is used).

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file. Variables are local to the `density` function.*

## Usage Examples

```c++
// Conceptual call (actual usage depends on many pre-initialized structures)
// TCSR<double> KS_matrix, S_matrix, P_matrix, PImag_matrix;
// CPX E_point, E_weight, dE_weight;
// transport_methods::transport_method_type current_method = transport_methods::NEGF;
// std::vector<double> chemical_potentials;
// std::vector<contact_type> contacts_info;
// std::vector<result_type> results_output;
// std::vector<int> block_sizes_for_rgf;
// std::vector<int> orbitals_per_atom_map;
// transport_parameters sim_params;
// MPI_Comm current_communicator = MPI_COMM_WORLD;

// // ... Initialize all the above structures and parameters ...

// int status = density(&KS_matrix, &S_matrix, &P_matrix, &PImag_matrix,
//                      E_point, E_weight, dE_weight, current_method,
//                      chemical_potentials, contacts_info, results_output,
//                      block_sizes_for_rgf, orbitals_per_atom_map,
//                      sim_params, current_communicator);
// if (status == 0) {
//     // Process results_output and P_matrix/PImag_matrix
// }
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <math.h>`
*   `#include "SuperLU.H"` (Conditionally if `HAVE_SUPERLU` is defined)
*   `#include <time.h>`
*   `#include <stdlib.h>`
*   `#include <stdio.h>`
*   `#include <limits>`
*   `#include <algorithm>`
*   `#include "ScaLapack.H"`
*   `#include "tmprGF.H"`
*   `#include "FullInvert.H"`
*   `#include "LinearSolver.H"`
*   `#include "Full.H"`
*   `#include "Banded.H"`
*   `#include "MUMPS.H"` (Conditionally if `HAVE_MUMPS` is defined)
*   `#include "Connection.H"`
*   `#include "SplitSolve.H"` (Conditionally if `HAVE_SPLITSOLVE` is defined)
*   `#include "c_pexsi_interface.h"` (Conditionally if `HAVE_PEXSI` is defined)
*   `#include "Pardiso.H"` (Conditionally if `HAVE_PARDISO_SELINV` is defined)
*   `#include "GetSigma.H"`
*   `#include "Density.H"`
*   `#include <iostream>`

### External Libraries
*   **MPI**: Used for parallel communication and coordination.
*   **BLAS/LAPACK**: Implied through `ScaLapack.H` and direct calls like `c_zscal`, `c_daxpy`, `c_zgemm`, `c_zdotc` (likely from `Blas.H`).
*   **ScaLAPACK**: Used for distributed linear algebra if certain solver paths are taken.
*   **SuperLU_DIST**: (Conditional) Used as a sparse direct solver if `HAVE_SUPERLU` is defined.
*   **MUMPS**: (Conditional) Used as a sparse direct solver if `HAVE_MUMPS` is defined.
*   **PEXSI**: (Conditional) Used for pole expansion and selected inversion if `HAVE_PEXSI` is defined.
*   **Pardiso**: (Conditional) Used as a sparse direct solver or for selected inversion if `HAVE_PARDISO_SELINV` is defined.

### Interactions with Other Project Components
*   This function is a high-level driver for transport calculations.
*   **`CSR.H`**: Uses `TCSR<double>` and `TCSR<CPX>` for sparse matrix representation.
*   **`Types.H` / `CSR.H`**: Uses `contact_type`, `result_type`, `transport_parameters`, and `transport_methods` enum.
*   **`GetSigma.H` (`BoundarySelfEnergy` class)**: Calls methods from `BoundarySelfEnergy` to calculate self-energies for contacts.
*   **Linear Solvers**: Instantiates and uses various linear solver objects inherited from `LinearSolver<T>` (e.g., `Full`, `SuperLU`, `MUMPS`, `Banded`, `SplitSolve`) and direct inversion methods (`FullInvert`, `tmprGF::sparse_invert`, Pardiso, PEXSI).
*   **`Blas.H`**: Utilizes BLAS routines for dense matrix/vector operations.
*   **`Connection.H`**: May use MPI utilities defined there, although direct MPI calls are also present.
*   The function updates `Ps` and `PsImag` matrices which are then used in the broader SCF cycle or for final quantity calculation.

---
*This documentation is auto-generated and may require further manual refinement.*
