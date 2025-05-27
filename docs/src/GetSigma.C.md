# Documentation for `src/GetSigma.C`

## Overview

This file implements the `BoundarySelfEnergy` class, which is crucial for quantum transport calculations within the OMEN project, particularly for methods like NEGF (Non-Equilibrium Green's Function) and Wave Function approaches. The primary responsibility of this class is to compute the boundary self-energies (often denoted as Sigma, &Sigma;) for the contacts (leads) of the simulated device. These self-energies effectively model the influence of semi-infinite leads attached to the device region.

The calculation of self-energies can be performed using different numerical techniques depending on the energy (real or complex) and the chosen method:
*   For real energies, eigenvalue-based methods (using `InjectionBeyn.H` or `InjectionIEV.H`) are typically employed to find propagating and evanescent modes.
*   For complex energies (e.g., on a contour for integration), direct matrix inversion or related techniques (`GetSigmaInv`) might be used.

The class also handles the extraction of necessary Hamiltonian blocks, MPI communication for distributed calculations, and storage of related quantities like the Gamma matrix (coupling matrix) and injection terms for wave function methods.

## Key Components

### Functions
*(These are member functions of the `BoundarySelfEnergy` class, declared in `GetSigma.H`)*
*   `BoundarySelfEnergy()`: Constructor. Initializes member pointers to `NULL` and default values.
*   `~BoundarySelfEnergy()`: Destructor. Checks if memory has been deallocated, throwing an exception if not (indicating potential resource leaks).
*   `void create_M_matrix(CPX *M, int NR, TCSR<CPX> **A, int bw, CPX z)`: Constructs a matrix `M` (typically for eigenvalue problems) from block Hamiltonian components `A` (representing `H_0, H_1, ..., H_bw` and `H_{-1}, ..., H_{-bw}`) and a complex energy `z` (used as `e^{ik_za}`).
*   `void Deallocate_Sigma()`: Frees memory allocated for the distributed sparse self-energy (`spsigmadist`).
*   `void Deallocate_Gamma()`: Frees memory allocated for the Gamma matrix (`gamma`).
*   `void Deallocate_Injection()`: Frees memory allocated for injection terms (`spainjdist`, `lambdapro`).
*   `void Finalize()`: Frees all major dynamically allocated memory associated with the self-energy calculation (`sigma`, `inj`, `gamma`).
*   `int Set_master(MPI_Comm matrix_comm, MPI_Comm boundary_comm)`: Determines and sets the `master_rank` for operations that are performed by a single process within the `boundary_comm` communicator.
*   `int Cutout(TCSR<CPX> *SumHamC, contact_type pcontact, CPX penergy, transport_methods::transport_method_type transport_method, MPI_Comm matrix_comm)`: Extracts the necessary Hamiltonian blocks (`H0`, `H1`, `H1t`) for a specific contact from the full device Hamiltonian (`SumHamC`). It sets flags `compute_inj` or `compute_gamma` based on the `transport_method`.
*   `void Distribute(TCSR<CPX> *SumHamC, MPI_Comm matrix_comm)`: Distributes the calculated full self-energy matrix `sigma` (which is computed on the master rank) into a sparse distributed format `spsigmadist`. Also broadcasts other results like number of propagating modes and condition numbers.
*   `int GetSigma(MPI_Comm boundary_comm, transport_parameters transport_params)`: Main driver function that decides whether to call `GetSigmaInv` (for complex energies if `transport_params.n_points_inv` is set) or `GetSigmaEig` (typically for real energies).
*   `int GetSigmaInv(MPI_Comm boundary_comm, transport_parameters transport_params)`: Calculates the self-energy using a method suitable for complex energies, often involving contour integration where `M(z)` is constructed and inverted/solved for different `z` points on the contour.
*   `int GetSigmaEig(MPI_Comm boundary_comm, transport_parameters transport_params)`: Calculates the self-energy using an eigenvalue-based approach (calling routines from `InjectionBeyn.H` or `InjectionIEV.H`). This method determines propagating and evanescent modes and constructs the self-energy from them.

### Classes/Structs
*   *None defined in this file. It implements methods for the `BoundarySelfEnergy` class declared in `GetSigma.H`.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual usage within a larger calculation (e.g., inside the 'density' function)
// BoundarySelfEnergy contact_self_energy;
// contact_type contact_params; // Initialized
// TCSR<CPX> device_hamiltonian; // Initialized
// CPX current_energy_point; // Initialized
// transport_methods::transport_method_type method = transport_methods::NEGF;
// transport_parameters sim_params; // Initialized
// MPI_Comm current_communicator = MPI_COMM_WORLD;
// MPI_Comm contact_communicator; // A sub-communicator for this contact

// contact_self_energy.Set_master(current_communicator, contact_communicator);
// if (contact_self_energy.Cutout(&device_hamiltonian, contact_params, current_energy_point, method, current_communicator) != 0) { /* error */ }
// if (contact_self_energy.GetSigma(contact_communicator, sim_params) != 0) { /* error */ }
// contact_self_energy.Distribute(&device_hamiltonian, current_communicator);
// // Now contact_self_energy.spsigmadist can be used.
// contact_self_energy.Deallocate_Sigma();
// contact_self_energy.Finalize(); // Or more specific deallocators
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <math.h>`
*   `#include <iostream>`
*   `#include <time.h>`
*   `#include <stdlib.h>`
*   `#include <stdio.h>`
*   `#include <valarray>`
*   `#include "ScaLapack.H"`
*   `#include "InjectionBeyn.H"`
*   `#include "InjectionIEV.H"`
*   `#include "GetSigma.H"`

### External Libraries
*   **MPI**: Used for parallel communication and coordination.
*   **ScaLAPACK**: Potentially used by the eigenvalue solvers in `InjectionBeyn.H` or `InjectionIEV.H`.
*   **BLAS/LAPACK**: Implied by usage of functions like `c_zlacpy`, `c_zaxpy`, `c_zgemm`, `c_zdotc`, `c_dscal`, `c_zgetrf`, `c_zgecon` (from `Blas.H`).

### Interactions with Other Project Components
*   This file implements the `BoundarySelfEnergy` class, declared in `GetSigma.H`.
*   **`CSR.H` (`TCSR<CPX>`)**: Extensively uses and manipulates sparse matrices in `TCSR` format.
*   **`Types.H` / `CSR.H`**: Uses `contact_type`, `transport_parameters`, `CPX`, and enums like `transport_methods`.
*   **`InjectionBeyn.H` / `InjectionIEV.H`**: Uses classes from these headers to solve generalized eigenvalue problems for mode calculations.
*   **`Blas.H`**: Uses BLAS and LAPACK wrapper functions for dense matrix operations.
*   The calculated self-energies are fundamental inputs for Green's function-based transport calculations performed by other modules (e.g., `Density.C`).

---
*This documentation is auto-generated and may require further manual refinement.*
