# Documentation for `src/SemiSelfConsistent.C`

## Overview

This file implements the `semiselfconsistent` function, which is central to performing a semi-self-consistent Schrödinger-Poisson simulation. It iteratively calculates the charge density (from a Schrödinger-like equation, likely using Green's functions or a similar method via `Energyvector::Execute`) and the electrostatic potential (via `OMEN_Poisson_Solver->solve`) until a consistent solution is achieved. This process is fundamental for simulating electronic devices where charge distribution and potential are interdependent.

The function handles initialization of the potential, the iterative self-consistency loop, and manages MPI communication for distributed calculations.

## Key Components

### Functions
*   `int semiselfconsistent(cp2k_csr_interop_type S, cp2k_csr_interop_type KS, cp2k_csr_interop_type *P, cp2k_csr_interop_type *PImag, std::vector<double> muvec, std::vector<contact_type> contactvec, std::vector<int> Bsizes, std::vector<int> orb_per_atom, double mixing_parameter, transport_parameters transport_params)`:
    *   **Description**: This is the main function in the file. It orchestrates the self-consistent iterations.
    *   **Parameters**:
        *   `S`, `KS`: CP2K CSR matrices (Overlap and Kohn-Sham like).
        *   `P`, `PImag`: Output CP2K CSR matrices (likely Density matrix components).
        *   `muvec`: Vector of chemical potentials for the contacts.
        *   `contactvec`: Vector of `contact_type` structs describing device contacts.
        *   `Bsizes`: Vector of block sizes.
        *   `orb_per_atom`: Vector detailing orbitals per atom.
        *   `mixing_parameter`: Mixing factor for updating charge density between iterations.
        *   `transport_params`: Structure containing various parameters for the transport calculation.
    *   **Returns**: `0` on success, non-zero on failure.

### Classes/Structs
*No classes or structs are defined within this file. It utilizes classes like `Singularities` and `Energyvector` defined in other headers.*

### Enums
*   `enum choose_initial_guess {ramp, ferm, file};`: Defines options for how the initial electrostatic potential guess is generated.

## Important Variables/Constants
*(These are local to the `semiselfconsistent` function but are critical for its operation)*
*   `Vnew`: (Type `double*`) Array storing the newly calculated electrostatic potential on the grid.
*   `Vold`: (Type `double*`) Array storing the electrostatic potential from the previous iteration.
*   `rho_atom`: (Type `double*`) Array storing the calculated charge density per atom.
*   `rho_atom_previous`: (Type `double*`) Array storing the mixed charge density from the previous iteration.
*   `atom_of_bf`: (Type `int*`) Maps each basis function index to its corresponding atom index.
*   `min_cond_band_edge`: (Type `double`) Minimum of the conduction band edge, used for aligning potentials.
*   `dV`, `mu_avg`, `Vs`, `Vd`, `Vg`: Variables related to applied voltages and potential alignment.
*   `initial_guess`: (Type `choose_initial_guess`) Determines the method for setting the initial potential profile.
*   `residual`: (Type `double`) Stores the residual of the Poisson solver.
*   `density_old`, `density_new`: (Type `double`) Total integrated density in previous and current iterations.
*   `density_criterion`: (Type `double`) Convergence criterion for the charge density.
*   `max_iter`: (Type `int`) Maximum number of self-consistent iterations.

## Usage Examples

```c++
// Conceptual call (actual usage depends on many pre-initialized structures)
// cp2k_csr_interop_type S_matrix, KS_matrix, P_matrix, PImag_matrix;
// std::vector<double> mu_levels;
// std::vector<contact_type> contacts;
// std::vector<int> block_sizes, orbitals_per_atom_map;
// double mix_param;
// transport_parameters trans_params;

// // ... Initialize all the above structures and parameters ...

// int result = semiselfconsistent(S_matrix, KS_matrix, &P_matrix, &PImag_matrix,
//                                 mu_levels, contacts, block_sizes, orbitals_per_atom_map,
//                                 mix_param, trans_params);
// if (result == 0) {
//     // Self-consistent solution found
// } else {
//     // Error occurred
// }
```

## Dependencies and Interactions

### Included Files
*   `#include "GetSingularities.H"`
*   `#include "EnergyVector.H"`
*   `#include <numeric>`        // For `std::accumulate`
*   `#include <iterator>`       // Potentially for iterator adaptors or types (though not directly visible)
*   `#include <limits>`         // For `std::numeric_limits`
*   `#include "SemiSelfConsistent.H"` // Corresponding header file

### External Libraries
*   **MPI**: Used for parallel processing (`MPI_Comm_rank`, `MPI_Comm_size`).
*   **BLAS**: Implied by the use of BLAS-like functions prefixed with `c_` (e.g., `c_dscal`, `c_dasum`, `c_dcopy`, `c_daxpy`), which are likely wrappers for BLAS routines.

### Interactions with Other Project Components
*   **`GetSingularities.H` / `Singularities` class**: Used to determine initial Fermi levels if not provided.
*   **`EnergyVector.H` / `Energyvector` class**: Called within the loop to solve the Schrödinger part and calculate charge density (`rho_atom`).
*   **`OMEN_Poisson_Solver`**: An external component (likely a class instance) used to solve the Poisson equation for the potential `Vnew`.
*   **CP2K Data Structures (`cp2k_csr_interop_type`)**: Uses and manipulates sparse matrices in CP2K's CSR interoperability format.
*   **Global/External Structs**: Reads from global-like structures such as `FEM`, `voltage`, `nanowire`, `parameter`, `Wire`.
*   **Input Files**: Can read initial potential from "potgrid.input" and Fermi levels from "OMEN_F".
*   **Output Files**: Writes intermediate potential grid ("potgridfile[iter]") and charge density ("rhofile[iter]") for debugging/monitoring.

---
*This documentation is auto-generated and may require further manual refinement.*
