# Documentation for `src/GetSingularities.C`

## Overview

This file implements the `Singularities` class. The primary purpose of this class is to analyze the electronic band structure of materials to find critical energy points, often referred to as singularities in the Density of States (DOS). These include:
*   The overall ground state energy (`energy_gs`).
*   Valence band maxima (`energies_vb`) and conduction band minima (`energies_cb`) for each contact/region.
*   Other band extrema (points where group velocity is zero), which are important for constructing accurate energy integration meshes for transport calculations.

The class calculates the band structure E(k) by solving the generalized eigenvalue problem (H - E*S)c = 0 for different k-points. It can also compute derivatives of E(k) to find velocities and curvatures. This information is then used to determine Fermi levels based on doping and to guide the selection of energy points for Green's function or NEGF calculations.

## Key Components

### Functions
*(These are member functions of the `Singularities` class, declared in `GetSingularities.H`)*
*   `Singularities(transport_parameters transport_params, std::vector<contact_type> pcontactvec)`: Constructor. Initializes parameters, MPI communicators for parallel k-point calculations, and resizes internal data structures.
*   `~Singularities()`: Destructor. Frees MPI communicators.
*   `int Execute(cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type Overlap)`: Main public method. It iterates through contacts and k-points, calls `determine_velocities` to get E(k) and its derivatives, and then processes this data to find band edges and other extrema.
*   `std::vector< std::vector< std::vector<double> > > get_propagating(const std::vector<CPX> &evec)`: Given a vector of energies `evec`, this function (primarily on rank 0) determines the k-values of propagating modes at these energies by analyzing the pre-calculated band structure.
*   `double determine_charge(double mu, int i_mu, int mode)`: Calculates the electronic charge for a given contact `i_mu` and Fermi level `mu`, by integrating the Fermi-Dirac distribution over the bands. `mode` can specify different integrands (e.g., for total charge or related quantities).
*   `double determine_free_charge(double mu, int i_mu)`: Calculates the free carrier concentration for a contact `i_mu` at a Fermi level `mu`.
*   `double determine_fermi(double n_ele, int i_mu)`: Calculates the Fermi level for contact `i_mu` that corresponds to a given number of electrons `n_ele`. It uses an iterative root-finding approach (bisection and Newton-Raphson).
*   `void follow_band(int i_mu)`: Attempts to sort energy bands across k-points to maintain a consistent ordering, which can be challenging due to band crossings.
*   `void write_bandstructure(int i_mu, int iter_num, int do_follow_band, int do_debug)`: Writes the calculated band structure (energies, derivatives, curvatures) to output files for a specific contact `i_mu`.
*   `int determine_imaginary_bandstructure(TCSR<double> **H, TCSR<double> **S, double kimag_in, CPX *lambda, int i_mu)`: Calculates the complex band structure E(k_real + i*k_imag) for a given imaginary component `kimag_in`. This is used for finding evanescent modes.
*   `int determine_velocities(TCSR<double> **H, TCSR<double> **S, double k_in, double *energies_k, double *derivatives_k, double *curvatures_k, int ndof, int bandwidth)`: For a given real k-point `k_in`, this function solves the generalized eigenvalue problem for a block Hamiltonian (constructed from `H` and `S` representing different unit cell interactions) to get energies `energies_k`. If `dothederivs` is true, it also computes first (`derivatives_k`) and second (`curvatures_k`) derivatives of energy with respect to k.
*   `void add_full_to_scaled_cp2k_csr(...)`: A utility function seemingly for specific matrix manipulations related to CP2K CSR format (appears similar to one in `c_scf.C`, its direct role here needs clarification).
*   `int DensityFromBS(cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type Overlap, cp2k_csr_interop_type* Density, std::vector<double> muvec)`: Calculates the density matrix directly by integrating contributions from the band structure, weighted by Fermi-Dirac distributions.
*   `int determine_density_from_bs(...)`: Helper function for `DensityFromBS` to compute contributions at a specific k-point.

### Classes/Structs
*   *None defined in this file. It implements methods for the `Singularities` class declared in `GetSingularities.H`.*

## Important Variables/Constants
*   *None defined globally in this file. Member variables store settings and results.*

## Usage Examples

```c++
// Conceptual usage:
// transport_parameters tp_params; // Initialized
// std::vector<contact_type> contacts_data; // Initialized
// Singularities band_analyzer(tp_params, contacts_data);
// cp2k_csr_interop_type H_matrix, S_matrix; // Initialized

// if (band_analyzer.Execute(H_matrix, S_matrix) == 0) {
//     double fermi_level_contact0 = band_analyzer.determine_fermi(num_electrons_c0, 0);
//     // Access band_analyzer.energy_gs, energies_vb, energies_cb etc.
// }
```

## Dependencies and Interactions

### Included Files
*   `#include "GetSingularities.H"`
*   `#include <valarray>`
*   `#include <limits>`
*   `#include "Utilities.H"`
*   `#include "ParallelEig.H"`

### External Libraries
*   **MPI**: Used for parallelizing k-point calculations and distributing data.
*   **BLAS/LAPACK**: Implied by `ParallelEig.H` (which likely wraps ScaLAPACK or other parallel eigensolvers) and direct matrix operations like `c_dscal`, `c_zgemv`, `c_zdotc`.

### Interactions with Other Project Components
*   This file implements the `Singularities` class, declared in `GetSingularities.H`.
*   **`CSR.H` / `Types.H`**: Uses `TCSR<double>` for matrix representation, `cp2k_csr_interop_type` for input matrices, and structures like `transport_parameters`, `contact_type`. Uses `CPX` for complex numbers.
*   **`Utilities.H`**: Uses utility functions like `fermi`.
*   **`ParallelEig.H`**: Uses the parallel eigensolver (`p_eig`) to solve the generalized eigenvalue problem Hc = ESc for band structure calculations.
*   The calculated band edges and singularities are crucial inputs for the `EnergyVector` class, which constructs the energy meshes for Green's function or NEGF calculations.
*   The `determine_fermi` method is used to align Fermi levels based on doping concentrations.

---
*This documentation is auto-generated and may require further manual refinement.*
