# Documentation for `src/GetSingularities.H`

## Overview

This header file defines the `Singularities` class. The primary purpose of this class is to analyze the electronic band structure of materials to identify critical energy points. These points, often where the density of states (DOS) exhibits singular behavior (like band edges or van Hove singularities), are crucial for constructing accurate and efficient energy integration meshes in quantum transport calculations.

The Doxygen comment `/**  \brief Get the points where the DOS is singular and determine the integration range \author Sascha A. Brueck */` succinctly describes its main function.

## Key Components

### Classes/Structs
*   `class Singularities`:
    *   **Description**: Analyzes band structure to find band edges, extrema, and determine Fermi levels.
    *   **Public Members (Functions)**:
        *   `Singularities(transport_parameters transport_params, std::vector<contact_type> pcontactvec)`: Constructor.
        *   `int Execute(cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type Overlap)`: Main method to perform the band structure analysis.
        *   `double determine_charge(double mu, int i_mu, int mode)`: Calculates charge for a given Fermi level.
        *   `double determine_free_charge(double mu, int i_mu)`: Calculates free charge carrier concentration.
        *   `double determine_fermi(double n_ele, int i_mu)`: Determines the Fermi level for a given electron count.
        *   `void write_bandstructure(int i_mu, int iter_num, int do_follow_band, int do_debug)`: Writes calculated band structure to files.
        *   `std::vector< std::vector< std::vector<double> > > get_propagating(const std::vector<CPX>& evec)`: Identifies k-values of propagating modes for given energies.
        *   `int DensityFromBS(cp2k_csr_interop_type KohnSham, cp2k_csr_interop_type Overlap, cp2k_csr_interop_type* Density, std::vector<double> muvec)`: Calculates density matrix directly from band structure.
        *   `~Singularities()`: Destructor.
    *   **Public Members (Variables)**:
        *   `std::vector< std::vector<double> > energies_extremum`: Stores the energies of band extrema.
        *   `double energy_gs`: Stores the global ground state energy found.
        *   `std::vector<double> energies_vb`: Stores valence band edge energies for each contact.
        *   `std::vector<double> energies_cb`: Stores conduction band edge energies for each contact.
    *   **Private Members (Variables)**:
        *   `int dothederivs`: Flag to control calculation of energy derivatives.
        *   `double eps_singularities`, `eps_mu`: Tolerance parameters.
        *   `int n_k`: Number of k-points for band structure calculation.
        *   `double Temp`: Temperature.
        *   `double evfac`: Energy conversion factor.
        *   `int n_mu`: Number of contacts/chemical potentials.
        *   `int iam, k_rank, size_bs_comm, rank_bs_comm`: MPI rank and communicator details.
        *   `MPI_Comm bs_comm, equal_bs_rank_comm`: MPI communicators for parallel k-point calculations.
        *   `std::vector<int> master_ranks`: Ranks responsible for specific k-point calculations.
        *   `std::vector<contact_type> contactvec`: Stores contact information.
        *   `std::vector< std::vector<double> > curvatures_extremum, kval_extremum, energies_matrix, derivatives_matrix, curvatures_matrix`: Store detailed band structure information (curvatures, k-values, energies, derivatives).
    *   **Private Members (Functions)**:
        *   `int determine_velocities(...)`: Calculates E(k) and its derivatives at a k-point.
        *   `int determine_imaginary_bandstructure(...)`: Calculates E(k) for imaginary k.
        *   `void follow_band(int i_mu)`: Attempts to sort/track bands.
        *   `void add_full_to_scaled_cp2k_csr(...)`: Utility for matrix manipulation.
        *   `int determine_density_from_bs(...)`: Helper for calculating density from band structure.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual:
// transport_parameters params;
// std::vector<contact_type> contacts;
// // ... initialize params and contacts ...
// Singularities singularity_finder(params, contacts);
// cp2k_csr_interop_type H, S;
// // ... initialize H and S matrices ...
// singularity_finder.Execute(H, S);
// double fermi_level = singularity_finder.determine_fermi(target_electron_count, 0);
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include <vector>`

### External Libraries
*   **MPI**: Implied by `MPI_Comm` members and usage in the corresponding `.C` file for parallel k-point calculations.

### Interactions with Other Project Components
*   This header declares the `Singularities` class, which is implemented in `GetSingularities.C`.
*   It depends on `CSR.H` for `TCSR<double>` (sparse matrix type), `cp2k_csr_interop_type` (CP2K matrix format), and various data structures like `transport_parameters`, `contact_type`, and `CPX` (complex number type, likely defined in `Types.H` and included via `CSR.H`).
*   The results produced by this class (band edges, DOS singularities, Fermi levels) are crucial for the `EnergyVector` module, which uses this information to construct appropriate energy integration meshes for transport calculations.

---
*This documentation is auto-generated and may require further manual refinement.*
