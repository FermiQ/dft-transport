# Documentation for `src/GetSigma.H`

## Overview

This header file defines the `BoundarySelfEnergy` class. This class is responsible for computing the boundary self-energies (&Sigma;) for contacts (leads) in a quantum transport simulation. Self-energies are crucial for modeling open boundary conditions, allowing the simulation of current flow through a device connected to semi-infinite reservoirs. The class also handles related quantities like the injection terms for wave function methods and Gamma matrices (coupling matrices).

The Doxygen comment `/**  \brief Compute Boundary Self Energies and Injection \author Sascha A. Brueck */` highlights its main purpose.

## Key Components

### Classes/Structs
*   `class BoundarySelfEnergy`:
    *   **Description**: Manages the calculation and storage of boundary self-energies and related terms for device contacts.
    *   **Public Members (Functions)**:
        *   `BoundarySelfEnergy()`: Constructor.
        *   `~BoundarySelfEnergy()`: Destructor.
        *   `void Deallocate_Sigma()`: Frees memory allocated for the distributed sparse self-energy.
        *   `void Deallocate_Gamma()`: Frees memory for the Gamma matrix.
        *   `void Deallocate_Injection()`: Frees memory for injection terms.
        *   `void Finalize()`: Frees all major allocated memory related to the self-energy calculation.
        *   `int Set_master(MPI_Comm matrix_comm, MPI_Comm boundary_comm)`: Sets the master MPI rank for operations within a specific boundary communicator.
        *   `int Cutout(TCSR<CPX>* SumHamC, contact_type pcontact, CPX penergy, transport_methods::transport_method_type transport_method, MPI_Comm matrix_comm)`: Extracts Hamiltonian blocks (`H0`, `H1`) relevant to a contact from the full device Hamiltonian.
        *   `void Distribute(TCSR<CPX>* SumHamC, MPI_Comm matrix_comm)`: Distributes the calculated full self-energy (`sigma`) into a sparse distributed format (`spsigmadist`) and broadcasts other results.
        *   `int GetSigma(MPI_Comm boundary_comm, transport_parameters transport_params)`: Main function to trigger self-energy calculation, choosing between eigenvalue or inversion methods based on energy.
        *   `int GetSigmaEig(MPI_Comm boundary_comm, transport_parameters transport_params)`: Calculates self-energy using an eigenvalue-based method.
        *   `int GetSigmaInv(MPI_Comm boundary_comm, transport_parameters transport_params)`: Calculates self-energy using a direct inversion-based method (typically for complex energies).
    *   **Public Members (Variables)**:
        *   `CPX *sigma`: Pointer to the calculated dense self-energy matrix (typically on the master rank before distribution).
        *   `CPX *gamma`: Pointer to the Gamma matrix (&Gamma; = i(&Sigma; - &Sigma;<sup>&dagger;</sup>)).
        *   `TCSR<CPX> *spsigmadist`: Pointer to the distributed sparse self-energy matrix.
        *   `CPX *inj`: Pointer to the dense injection matrix (for wave function methods).
        *   `TCSR<CPX> *spainjdist`: Pointer to the distributed sparse injection matrix.
        *   `int n_propagating`: Number of propagating modes.
        *   `CPX *lambdapro`: Pointer to the phase factors (eigenvalues) of propagating modes.
        *   `double rcond`: Reciprocal condition number from eigenvalue or matrix inversion steps.
        *   `int n_dec`: Number of decaying modes.
        *   `int eigval_degeneracy`: Flag or index indicating eigenvalue degeneracy issues.
    *   **Private Members (Functions)**:
        *   `void create_M_matrix(CPX* M, int NR, TCSR<CPX>** A, int bw, CPX z)`: Constructs a matrix used in some self-energy calculation schemes.
    *   **Private Members (Variables)**:
        *   `int master_rank`: MPI rank designated to perform master tasks for a given boundary.
        *   `CPX energy`: The complex energy at which the self-energy is calculated.
        *   `int compute_inj`: Flag to indicate if injection terms should be computed.
        *   `int compute_gamma`: Flag to indicate if Gamma matrices should be computed.
        *   `contact_type contact`: Stores parameters for the current contact.
        *   `TCSR<CPX> *H0`: Hamiltonian block H<sub>00</sub> for the contact.
        *   `TCSR<CPX> *H1`: Hamiltonian block H<sub>01</sub> (or H<sub>10</sub>) coupling to the lead.
        *   `TCSR<CPX> *H1t`: Transpose of H1.
        *   `TCSR<CPX> **H`: Array of Hamiltonian blocks (H<sub>0</sub>, H<sub>&plusmn;1</sub>, ..., H<sub>&plusmn;bw</sub>) for eigenvalue-based methods.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual usage:
// BoundarySelfEnergy self_energy_calculator;
// // ... (Initialize parameters, SumHamC, contact_info, energy_val, etc.) ...
// self_energy_calculator.Set_master(world_comm, contact_specific_comm);
// self_energy_calculator.Cutout(SumHamC, contact_info, energy_val, method_type, world_comm);
// self_energy_calculator.GetSigma(contact_specific_comm, transport_params_obj);
// self_energy_calculator.Distribute(SumHamC, world_comm);
// // Now self_energy_calculator.spsigmadist can be used by other parts of the code.
// // ... later, cleanup ...
// self_energy_calculator.Deallocate_Sigma();
// self_energy_calculator.Finalize();
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`

### External Libraries
*   **MPI**: Implied by `MPI_Comm` in function signatures and usage in the corresponding `.C` file.

### Interactions with Other Project Components
*   This header file declares the `BoundarySelfEnergy` class, which is implemented in `GetSigma.C`.
*   It is a fundamental component for NEGF, GF, and WF transport calculations, providing the self-energies that are used by the `Density` module.
*   It depends on `CSR.H` for the `TCSR<CPX>` sparse matrix type and various enums and structs like `contact_type`, `transport_methods::transport_method_type`, and `transport_parameters` (which are defined in or included via `CSR.H` or `Types.H`).

---
*This documentation is auto-generated and may require further manual refinement.*
