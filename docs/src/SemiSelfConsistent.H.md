# Documentation for `src/SemiSelfConsistent.H`

## Overview

This header file provides the declaration for the `semiselfconsistent` function, which is responsible for computing charge density and potentially transmission characteristics as part of a semi-self-consistent Schrödinger-Poisson simulation loop. The Doxygen comment `/**  \brief Compute density and transmission for a given energy \author Sascha A. Brueck */` suggests its core role. It also declares several `extern` global variables, indicating that the corresponding `.C` file relies on these global objects for its operation.

## Key Components

### Functions
*   `int semiselfconsistent(cp2k_csr_interop_type S, cp2k_csr_interop_type KS, cp2k_csr_interop_type *P, cp2k_csr_interop_type *PImag, std::vector<double> muvec, std::vector<contact_type> contactvec, std::vector<int> Bsizes, std::vector<int> orb_per_atom, double mixing_parameter, transport_parameters transport_params);`
    *   **Description**: This is the function declaration for the main semi-self-consistent calculation routine, implemented in `SemiSelfConsistent.C`. It takes various matrices, contact information, simulation parameters, and outputs density matrices.

### Classes/Structs
*No classes or structs are defined in this file.*

## Important Variables/Constants
*(These are `extern` declarations, meaning they are defined elsewhere, likely in global scope in another `.C` file within the project)*
*   `extern WireStructure *nanowire;`: Pointer to a `WireStructure` object, likely containing geometric and material properties of the simulated nanowire.
*   `extern WireGenerator* Wire;`: Pointer to a `WireGenerator` object, possibly used for creating or managing the wire structure.
*   `extern Poisson *OMEN_Poisson_Solver;`: Pointer to a `Poisson` solver object, used to solve the Poisson equation.
*   `extern FEMGrid *FEM;`: Pointer to an `FEMGrid` object, representing the finite element method grid.
*   `extern PARAM *parameter;`: Pointer to a `PARAM` object, likely holding various simulation parameters.
*   `extern VOLTAGE *voltage;`: Pointer to a `VOLTAGE` object, likely managing applied voltages and boundary conditions.

## Usage Examples

```c++
// Conceptual inclusion and usage (actual call is complex and relies on initialized data)
// #include "SemiSelfConsistent.H"

// // ... Assuming all parameters and extern variables are properly set up ...
// // cp2k_csr_interop_type S_matrix, KS_matrix, P_matrix, PImag_matrix;
// // std::vector<double> mu_levels;
// // std::vector<contact_type> contacts;
// // ... other parameters ...

// // int result = semiselfconsistent(S_matrix, KS_matrix, &P_matrix, &PImag_matrix,
// //                                 mu_levels, contacts, ...);
```

## Dependencies and Interactions

### Included Files
*   `#include "FEMGrid.H"`
*   `#include "Poisson.H"`
*   `#include <vector>`

### External Libraries
*No external libraries are directly included via their own headers in this file. However, the types used (e.g., `cp2k_csr_interop_type`) suggest dependencies on other parts of the OMEN code base or related libraries like CP2K.*

### Interactions with Other Project Components
*   This header file is crucial for any part of the OMEN project that needs to invoke the semi-self-consistent calculation loop.
*   It establishes a dependency on the components that define `WireStructure`, `WireGenerator`, `Poisson`, `FEMGrid`, `PARAM`, and `VOLTAGE` types, as well as `cp2k_csr_interop_type`, `contact_type`, and `transport_parameters`.
*   The corresponding `SemiSelfConsistent.C` file will implement the function and utilize these external global variables.

---
*This documentation is auto-generated and may require further manual refinement.*
