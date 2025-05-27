# Documentation for `src/EnergyVector.C`

## Overview

This file implements the `Energyvector` class. The primary responsibility of this class is to manage the energy-dependent aspects of quantum transport calculations. This involves:
1.  Determining an appropriate set of complex and/or real energy points and corresponding weights for numerical integration. This is crucial for calculating quantities like the density matrix or transmission. The energy points can be determined based on singularities in the Green's function (via the `Singularities` class), Fermi levels, temperature, and the chosen integration scheme (e.g., PEXSI pole expansion, Gauss-Chebyshev quadrature, trapezoidal rule).
2.  Orchestrating the execution of the core `density` calculation (from `Density.H`) over these selected energy points. This often involves distributing the workload across multiple MPI processes.
3.  Aggregating results from individual energy points, such as transmission, and writing them to output files.

## Key Components

### Functions
*(These are member functions of the `Energyvector` class, declared in `EnergyVector.H`)*
*   `Energyvector()`: Constructor. Initializes MPI rank and size.
*   `~Energyvector()`: Destructor.
*   `int Execute(cp2k_csr_interop_type Overlap, cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type *P, cp2k_csr_interop_type *PImag, std::vector<double> &muvec, std::vector<contact_type> contactvec, std::vector<int> Bsizes, std::vector<int> orb_per_at, double *rho_atom, transport_parameters transport_params)`:
    *   **Description**: The main public method that drives the energy integration process. It first calls `determine_energyvector` to get the energy points and weights, then calls `distribute_and_execute` to perform the calculations.
*   `int distribute_and_execute(std::vector<CPX> energyvector, std::vector<CPX> stepvector, std::vector<CPX> drdmvector, std::vector<CPX> energyvector_real, std::vector<CPX> stepvector_real, std::vector< std::vector<int> > propagating_sizes, distribution_methods::distribution_method_type distribution_method, int tasks_per_point, cp2k_csr_interop_type Overlap, cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type *P, cp2k_csr_interop_type *PImag, std::vector<double> &muvec, std::vector<contact_type> contactvec, std::vector<int> Bsizes, std::vector<int> orb_per_at, double *rho_atom, transport_parameters transport_params)`:
    *   **Description**: Manages the distribution of energy points among MPI processes and calls the `density` function for each assigned energy point. It handles the setup of MPI communicators for matrix distribution and collects results.
*   `std::vector<int> get_tsizes(distribution_methods::distribution_method_type distribution_method, int nrows_total_cut, std::vector<int> Bsizes, std::vector<int> orb_per_at, int gpus_per_point, int tasks_per_point)`:
    *   **Description**: Determines how to distribute the rows of matrices among MPI tasks based on the chosen distribution method and available resources.
*   `int write_transmission_current(std::vector<CPX> energyvector, std::vector<CPX> stepvector, std::vector<double> transmission, std::vector<double> muvec, transport_parameters transport_params)`:
    *   **Description**: Writes the calculated transmission spectrum and integrated current to output files.
*   `int determine_energyvector(std::vector<CPX> &energyvector_cc, std::vector<CPX> &stepvector_cc, std::vector<CPX> &drdmvector_cc, std::vector<CPX> &energyvector, std::vector<CPX> &stepvector, std::vector< std::vector<int> > &propagating_sizes, cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type Overlap, std::vector<double> &muvec, std::vector<contact_type> contactvec, transport_parameters transport_params)`:
    *   **Description**: Core logic for selecting the energy points for integration. It uses the `Singularities` class to find band edges and other critical points, then calls `assign_real_axis_energies` and/or `assign_cmpx_cont_energies` based on the simulation parameters. It also determines the number of propagating modes if needed.
*   `int assign_real_axis_energies(double nonequi_start, double nonequi_end, std::vector<CPX> &energyvector, std::vector<CPX> &stepvector, const std::vector< std::vector<double> > &energies_extremum, int muvec_size, transport_parameters transport_params)`:
    *   **Description**: Generates energy points and weights for real-axis integration using methods like trapezoidal rule or Gauss-Chebyshev quadrature, potentially adapting the mesh around band extrema.
*   `int assign_cmpx_cont_energies(double start, double mu, std::vector<CPX> &energyvector, std::vector<CPX> &stepvector, std::vector<CPX> &drdmvector, double Temp, int num_points_on_contour)`:
    *   **Description**: Generates energy points and weights for complex contour integration, often using pole expansion techniques (e.g., PEXSI).

### Classes/Structs
*   *None defined in this file. This file implements methods of the `Energyvector` class declared in `EnergyVector.H`.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Energyvector is typically used internally by higher-level simulation drivers.
// Conceptual usage:
// Energyvector energy_integrator;
// // ... (S, KS, P, PImag, muvec, contactvec, Bsizes, orb_per_at, rho_atom, transport_params
// //      are all initialized appropriately) ...
// int status = energy_integrator.Execute(S_matrix, KS_matrix, &P_matrix, &PImag_matrix,
//                                        mu_levels, contacts, block_sizes_rgf,
//                                        orbitals_map, charge_density_output, sim_params);
```

## Dependencies and Interactions

### Included Files
*   `#include "pexsi/pole.hpp"`
*   `#include "Utilities.H"`
*   `#include "Density.H"`
*   `#include "GetSingularities.H"`
*   `#include "Quadrature.H"`
*   `#include "EnergyVector.H"`
*   `#include <iterator>`
*   `#include <limits>`
*   `#include <vector>`

### External Libraries
*   **MPI**: Used extensively for distributing calculations and data.
*   **PEXSI**: Interacts with the PEXSI library for pole expansion (`pexsi/pole.hpp`, `PEXSI::GetPoleDensity`).

### Interactions with Other Project Components
*   **`EnergyVector.H`**: Implements the class declared in this header.
*   **`Density.H` (`density` function)**: The `Energyvector` class orchestrates calls to the `density` function for each selected energy point.
*   **`GetSingularities.H` (`Singularities` class)**: Uses the `Singularities` class to determine band edges and other critical energies for mesh generation.
*   **`Quadrature.H` (`Quadrature` class)**: Uses the `Quadrature` class to generate abscissae and weights for numerical integration schemes.
*   **`CSR.H`**: Works with `TCSR` sparse matrix types and `cp2k_csr_interop_type`. It also uses enums and structs like `transport_parameters`, `contact_type` defined or included via `CSR.H` or `Types.H`.
*   **`Utilities.H`**: Uses utility functions like `get_time`, `LOGCERR`.
*   **`Blas.H`**: Uses BLAS wrappers like `c_dcopy`, `c_dscal`, `c_daxpy`.

---
*This documentation is auto-generated and may require further manual refinement.*
