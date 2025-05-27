# Documentation for `src/Quadrature.C`

## Overview

This file implements the `Quadrature` class. The primary purpose of this class is to generate sets of abscissae (integration points) and corresponding weights for various numerical quadrature schemes. These schemes are used to approximate definite integrals, a common task in scientific computing, such as integrating functions over energy ranges or k-space in the OMEN project.

The constructor of the `Quadrature` class takes the desired quadrature type, the integration range (start and end points), and the number of abscissae, then calculates and stores the points and weights.

## Key Components

### Structs
*   `struct QUADRATURE_Exception`:
    *   **Description**: A simple exception structure used to report errors encountered during quadrature generation (e.g., invalid parameters). It logs the line number and file where the error occurred.

### Functions
*(This file primarily implements the constructor of the `Quadrature` class, which contains the logic for different quadrature rules.)*
*   `Quadrature::Quadrature(quadrature_types::quadrature_type type, double band_start, double band_end, int num_abscissae)`:
    *   **Description**: Constructor for the `Quadrature` class. It calculates and stores the abscissae and weights based on the specified quadrature `type`.
    *   **Doxygen Comment**: `/*! \brief Constructor ... Currently implemented methods for quadrature: 'quadrature_type::GC' ... */`
    *   **Implemented Quadrature Types**:
        *   `quadrature_types::NONE`: No quadrature is performed if parameters are invalid.
        *   `quadrature_types::CCGL` (Clenshaw-Curtis Gauss-Legendre?): For integration on a transformed circular contour. Involves solving an eigenvalue problem (`c_dsteqr`) for a companion matrix.
        *   `quadrature_types::GL` (Gauss-Legendre): Standard Gauss-Legendre quadrature for finite intervals. Also uses `c_dsteqr`.
        *   `quadrature_types::GC` (Gauss-Chebyshev): For integrals with a specific weight function, or for functions with endpoint singularities. Abscissae and weights are calculated using analytical formulas.
        *   `quadrature_types::TS` (Tanh-Sinh): A quadrature rule effective for functions with endpoint singularities or rapid variations.
        *   `quadrature_types::TR` (Trapezoidal Rule): A basic equally spaced quadrature rule.
        *   `quadrature_types::MR` (Midpoint Rule): Another basic equally spaced quadrature rule.
        *   `quadrature_types::CCMR` (Clenshaw-Curtis Midpoint Rule?): Similar to CCGL but possibly using midpoints on the transformed contour.

### Classes/Structs
*   *None defined in this file beyond `QUADRATURE_Exception`. It implements the `Quadrature` class constructor.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for using the Quadrature class
// double start_energy = -5.0;
// double end_energy = 5.0;
// int num_points = 64;
// Quadrature my_quadrature(quadrature_types::GC, start_energy, end_energy, num_points);

// // Access abscissae and weights:
// // for (size_t i = 0; i < my_quadrature.abscissae.size(); ++i) {
// //     CPX point = my_quadrature.abscissae[i];
// //     CPX weight = my_quadrature.weights[i];
// //     // ... use point and weight for integration ...
// // }
```

## Dependencies and Interactions

### Included Files
*   `#include <math.h>`
*   `#include "Quadrature.H"`
*   `#include "Blas.H"`

### External Libraries
*   **BLAS/LAPACK**: Implied by the use of `c_dsteqr` (likely a wrapper for `DSTEQR` from LAPACK, used for tridiagonal eigenvalue problems) via `Blas.H`.

### Interactions with Other Project Components
*   This file implements the constructor of the `Quadrature` class, which is declared in `Quadrature.H`.
*   It uses the `quadrature_types::quadrature_type` enum, likely defined in `Quadrature.H` or `CSR.H`.
*   The `CPX` type (from `Types.H`, included via `Quadrature.H` or `Blas.H`) is used for complex abscissae/weights in contour integrations.
*   The `Quadrature` class is used by other modules, notably `EnergyVector.C`, to generate energy points and weights for numerical integration of physical quantities.

---
*This documentation is auto-generated and may require further manual refinement.*
