# Documentation for `src/InjectionIEV.H`

## Overview

This header file defines the `InjectionIEV` class. This class appears to implement a method for calculating injection modes and their properties (eigenvalues, eigenvectors) for device contacts. The name suggests it might use an Iterative Eigenvalue (IEV) approach or a standard eigenvalue decomposition approach to solve the problem that determines these modes. This is an alternative to Beyn's contour integral method implemented in `InjectionBeyn.H`.

The results of this calculation (modes and their characteristics) are essential for constructing self-energies in quantum transport simulations.

## Key Components

### Classes/Structs
*   `class InjectionIEV`:
    *   **Description**: Calculates injection modes and related properties using an eigenvalue-based method.
    *   **Public Members (Functions)**:
        *   `InjectionIEV()`: Constructor.
        *   `~InjectionIEV()`: Destructor.
        *   `int Compute(int& nindnzcoln, CPX* eigvalcpx, CPX* eigvecc, CPX* KScpx, int ndof, int bandwidth, int inj_sign, int complexenergypoint, double colzerothr, MPI_Comm boundary_comm)`:
            *   **Description**: The main computational routine. It takes Hamiltonian components (implicitly within `KScpx` which is constructed from `H[0...2*bandwidth]`), solves a matrix eigenvalue problem to find the modes (`eigvalcpx`, `eigvecc`), and filters them.
            *   **Parameters**:
                *   `nindnzcoln`: (Output) Number of valid/non-zero column eigenvectors found.
                *   `eigvalcpx`: (Output) Array to store computed eigenvalues (k-phases).
                *   `eigvecc`: (Output) Array to store computed eigenvectors.
                *   `KScpx`: (Input) Pointer to an array containing the block matrix elements for the eigenvalue problem.
                *   `ndof`: Number of degrees of freedom per layer/principal block.
                *   `bandwidth`: Number of interacting layers/blocks in one direction.
                *   `inj_sign`: Injection sign (+1 or -1), indicating direction.
                *   `complexenergypoint`: Flag indicating if the calculation is at a complex energy.
                *   `colzerothr`: Threshold for considering a column as zero.
                *   `boundary_comm`: MPI communicator for this boundary calculation.
            *   **Returns**: An integer status code (0 for success).

### Functions
*   *None defined outside the class.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for InjectionIEV
// InjectionIEV iev_calculator;
// int num_valid_modes;
// CPX eigenvalues[MAX_MODES];
// CPX eigenvectors[DOF_PER_LAYER * MAX_MODES];
// CPX hamiltonian_blocks_for_gevp[...]; // Constructed from H0, H1 etc.
// int degrees_freedom = ...;
// int interaction_range = ...;
// int injection_direction = 1;
// int is_complex_energy = 0;
// double zero_threshold = 1e-9;
// MPI_Comm current_communicator = MPI_COMM_WORLD;

// int status = iev_calculator.Compute(num_valid_modes, eigenvalues, eigenvectors,
//                                     hamiltonian_blocks_for_gevp, degrees_freedom,
//                                     interaction_range, injection_direction,
//                                     is_complex_energy, zero_threshold,
//                                     current_communicator);
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <math.h>`
*   `#include <iostream>`
*   `#include <time.h>`
*   `#include <stdlib.h>`
*   `#include <stdio.h>`
*   `#include <limits>`
*   `#include <valarray>`
*   `#include "ScaLapack.H"`
*   `#include "GetSigma.H"`

### External Libraries
*   **MPI**: Used for parallel execution, indicated by `MPI_Comm`.
*   **ScaLAPACK**: Included via `ScaLapack.H`, suggesting potential use of distributed linear algebra routines, although direct calls are not in this header (likely in the `.C` file or through other functions).
*   **BLAS/LAPACK**: Implied by the use of routines like `c_zgetrf`, `c_dsysv`, `c_zgeev`, `c_dgeev`, `c_zgemm`, `c_zdotc` within the `Compute` method's implementation in the `.C` file.

### Interactions with Other Project Components
*   This class provides an alternative mechanism for calculating injection terms compared to `InjectionBeyn.H`.
*   The `Compute` method is the primary interface and is likely called by the `GetSigma` module when an eigenvalue-based approach is selected for self-energy calculation.
*   It depends on `Types.H` (via `GetSigma.H` or `ScaLapack.H`) for the `CPX` complex data type.
*   It uses BLAS/LAPACK wrappers (likely from `Blas.H`, included via other headers) for its dense matrix operations.

---
*This documentation is auto-generated and may require further manual refinement.*
