# Documentation for `src/Density.H`

## Overview

This header file declares the main `density` function, which is a central part of the OMEN project's capability to compute quantum transport properties. The function is responsible for calculating physical quantities such as density matrices, density of states (DOS), and transmission probabilities for a given energy point.

The Doxygen comment `/**  \brief Compute density and transmission for a given energy \author Sascha A. Brueck */` succinctly describes its primary purpose.

## Key Components

### Functions
*   `int density(TCSR<double> *KohnSham, TCSR<double> *Overlap, TCSR<double> *Ps, TCSR<double> *PsImag, CPX energy, CPX weight, CPX dweight, transport_methods::transport_method_type method, std::vector<double> muvec, std::vector<contact_type> contactvec, std::vector<result_type> &resultvec, std::vector<int> Bsizes, std::vector<int> orb_per_at, transport_parameters transport_params, MPI_Comm matrix_comm);`
    *   **Description**: This is the main function declaration for the routine that computes density and transmission characteristics. It takes Kohn-Sham and Overlap matrices, output density matrices, energy and weight parameters, the transport method to be used, contact information, and other simulation parameters.
    *   **Parameters**:
        *   `KohnSham`: Pointer to the Kohn-Sham matrix (real, CSR format).
        *   `Overlap`: Pointer to the Overlap matrix (real, CSR format).
        *   `Ps`: Pointer to the output real part of the density matrix (CSR format).
        *   `PsImag`: Pointer to the output imaginary part of the density matrix (CSR format).
        *   `energy`: Complex energy point (E + i*eta).
        *   `weight`: Weight of the energy point for integration.
        *   `dweight`: Derivative of the weight (for Fermi function derivative).
        *   `method`: Enum specifying the transport calculation method (e.g., NEGF, GF).
        *   `muvec`: Vector of chemical potentials for contacts.
        *   `contactvec`: Vector of `contact_type` structs.
        *   `resultvec`: Reference to a vector to store results (transmission, DOS).
        *   `Bsizes`: Vector of block sizes for block-based solvers.
        *   `orb_per_at`: Vector mapping atoms to orbital counts.
        *   `transport_params`: Struct with various transport simulation parameters.
        *   `matrix_comm`: MPI communicator for matrix operations.
    *   **Returns**: An integer status code (0 for success).

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined in this file.*

## Usage Examples

```c++
// Conceptual call (actual usage is complex and relies on many pre-initialized OMEN structures)
// int status = density(pKohnSham, pOverlap, pDensityMatrixReal, pDensityMatrixImag,
//                      current_energy, current_weight, current_dweight,
//                      selected_method, mu_values, contact_definitions,
//                      output_results, block_config, orbital_map,
//                      transport_settings, mpi_communicator);
```

## Dependencies and Interactions

### Included Files
*   `#include <numeric>`
*   `#include <vector>`
*   `#include "CSR.H"`

### External Libraries
*   **MPI**: Implied by the `MPI_Comm` parameter.

### Interactions with Other Project Components
*   This header file declares the interface for the `density` function, which is implemented in `Density.C`.
*   It depends on `CSR.H` for several key definitions:
    *   `TCSR<double>`: The sparse matrix type used for input and output matrices.
    *   `CPX`: The complex number type (likely defined in `Types.H` but included via `CSR.H`).
    *   `transport_methods::transport_method_type`: Enum for selecting the calculation method.
    *   `contact_type`: Struct for defining contact properties.
    *   `result_type`: Struct for storing calculation results.
    *   `transport_parameters`: Struct for passing various simulation settings.
*   The `density` function is a high-level routine that orchestrates calls to various solver and self-energy modules.

---
*This documentation is auto-generated and may require further manual refinement.*
