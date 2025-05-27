# Documentation for `src/Quadrature.H`

## Overview

This header file defines the `Quadrature` class and an enumeration `quadrature_types::quadrature_type`. The `Quadrature` class is designed to provide abscissae (integration points) and corresponding weights for various numerical quadrature schemes. These schemes are essential for approximating definite integrals, a common requirement in the OMEN project, for example, when integrating functions over energy.

The Doxygen comment `/*! \class Quadrature \brief Provides access to abscissae and weights for quadratures ... */` clearly states the purpose of the class.

## Key Components

### Namespaces
*   `quadrature_types`:
    *   **Enums**:
        *   `enum quadrature_type`: Defines the different types of quadrature rules available.
            *   `NONE`: No quadrature / placeholder.
            *   `CCGL`: Complex Contour Gauss-Legendre.
            *   `GL`: Gauss-Legendre.
            *   `GC`: Gauss-Chebyshev.
            *   `TS`: Tanh-Sinh.
            *   `TR`: Trapezoidal Rule.
            *   `CCMR`: Complex Contour Midpoint Rule.
            *   `MR`: Midpoint Rule.

### Classes/Structs
*   `class Quadrature`:
    *   **Description**: Provides access to abscissae and weights for numerical quadrature. The constructor calculates these based on the chosen type.
    *   **Public Members (Variables)**:
        *   `std::vector<CPX> abscissae`: Stores the calculated quadrature points (can be complex for contour integrations).
        *   `std::vector<CPX> weights`: Stores the corresponding quadrature weights (can be complex).
    *   **Public Members (Functions)**:
        *   `Quadrature(quadrature_types::quadrature_type type, double band_start, double band_end, int num_abscissae)`: Constructor that takes the quadrature type, integration range (`band_start`, `band_end`), and the desired number of abscissae.

### Functions
*   *None defined outside the class.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for Quadrature class
// Quadrature quad_rule(quadrature_types::GC, -1.0, 1.0, 32);
// for (size_t i = 0; i < quad_rule.abscissae.size(); ++i) {
//    CPX point = quad_rule.abscissae[i];
//    CPX weight = quad_rule.weights[i];
//    // Perform integration step: result += function(point) * weight;
// }
```

## Dependencies and Interactions

### Included Files
*   `#include "Types.H"`
*   `#include <vector>`

### External Libraries
*   *None directly identifiable from this header file. The implementation in `Quadrature.C` might use external libraries like BLAS/LAPACK for some schemes.*

### Interactions with Other Project Components
*   This header file declares the `Quadrature` class, which is implemented in `Quadrature.C`.
*   The `quadrature_types::quadrature_type` enum is used to select the integration method.
*   It depends on `Types.H` for the `CPX` complex number type.
*   The `Quadrature` class is utilized by other modules within OMEN, such as `EnergyVector.C`, to generate energy points and weights needed for numerical integration of physical quantities (e.g., density of states, Green's functions).

---
*This documentation is auto-generated and may require further manual refinement.*
