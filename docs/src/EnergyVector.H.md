# Documentation for `src/EnergyVector.H`

## Overview

This header file defines the `Energyvector` class, which plays a crucial role in managing and executing energy-dependent calculations within the OMEN project. Its primary responsibilities include determining the set of energy points required for numerical integration (e.g., for density matrix calculation or transmission spectra), distributing these calculations across MPI processes, and invoking the core `density` function for each energy point.

The Doxygen comment `/*!  \brief Distribute matrices from CP2K to the energy points, call bandstructure calculator, construct energyvector and call density evaluation in parallel \author Sascha A. Brueck */` highlights its function in coordinating energy-resolved computations.

This file also defines the `distribution_methods` namespace containing an enumeration for different strategies to distribute matrix data among MPI processes.

## Key Components

### Namespaces
*   `distribution_methods`:
    *   **Enums**:
        *   `enum distribution_method_type`: Defines strategies for distributing matrix data across MPI processes. Values include:
            *   `NO_DISTRIBUTION`
            *   `SPLITSOLVE_DISTRIBUTION`
            *   `FLOOR_DISTRIBUTION`
            *   `CEILING_DISTRIBUTION`
            *   `MASTER_DISTRIBUTION`

### Classes/Structs
*   `class Energyvector`:
    *   **Description**: Orchestrates calculations over a vector of energy points, including determining the energy mesh and distributing tasks.
    *   **Public Members (Functions)**:
        *   `Energyvector()`: Constructor.
        *   `int Execute(cp2k_csr_interop_type Overlap, cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type *P, cp2k_csr_interop_type *PImag, std::vector<double> &muvec, std::vector<contact_type> contactvec, std::vector<int> Bsizes, std::vector<int> orb_per_at, double *rho_atom, transport_parameters transport_params)`: The main method to initiate the energy-dependent calculation.
        *   `~Energyvector()`: Destructor.
    *   **Private Members (Functions)**:
        *   `int distribute_and_execute(...)`: Manages the parallel execution of the `density` function over the energy points.
        *   `std::vector<int> get_tsizes(...)`: Determines the size of matrix chunks for each MPI task based on the distribution strategy.
        *   `int write_transmission_current(...)`: Writes calculated transmission and current to a file.
        *   `int determine_energyvector(...)`: Determines the energy points and integration weights.
        *   `int assign_real_axis_energies(...)`: Logic for generating energy points along the real axis.
        *   `int assign_cmpx_cont_energies(...)`: Logic for generating energy points along a complex contour.
    *   **Private Members (Variables)**:
        *   `int iam`: MPI rank of the current process.
        *   `int nprocs`: Total number of MPI processes.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual usage:
// Energyvector calculator;
// // ... (Initialize all required parameters for Execute method) ...
// int result = calculator.Execute(S_matrix, KS_matrix, &P_out, &PImag_out,
//                                 fermi_levels, contacts, block_sizes,
//                                 orbitals_info, atom_charges, transport_params_obj);
```

## Dependencies and Interactions

### Included Files
*   `#include "libcp2k.h"`
*   `#include <vector>`
*   Implicit dependencies on `CSR.H` or `Types.H` for `cp2k_csr_interop_type`, `contact_type`, `CPX`, and `transport_parameters` through function signatures.

### External Libraries
*   **CP2K**: Indicated by `libcp2k.h`, suggesting interaction with CP2K data structures.
*   **MPI**: Implied by member variables `iam`, `nprocs` and usage in the corresponding `.C` file.

### Interactions with Other Project Components
*   This header declares the `Energyvector` class, which is implemented in `EnergyVector.C`.
*   The `Energyvector::Execute` method is a high-level driver that calls the `density` function (declared in `Density.H`).
*   It uses various data structures likely defined in `Types.H` and `CSR.H` (e.g., `cp2k_csr_interop_type`, `contact_type`, `transport_parameters`, `CPX`).
*   The `distribution_methods` enum provides strategies for how matrix data is parallelized across MPI ranks when calling lower-level routines.

---
*This documentation is auto-generated and may require further manual refinement.*
