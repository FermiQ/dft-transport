# Documentation for `src/Fermi.H`

## Overview

This header file defines the `Fermi` class. The purpose of this class is to calculate the Fermi level (chemical potential) for a given semiconductor material. This calculation typically involves using the material's density of states and doping concentration to solve the charge neutrality equation.

## Key Components

### Classes/Structs
*   `class Fermi`:
    *   **Description**: A class designed to determine the Fermi level.
    *   **Public Members (Functions)**:
        *   `Fermi()`: Constructor.
        *   `~Fermi()`: Destructor.
        *   `double find_fermi(double Ndop, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double spin_factor, double Temp, double A)`: The main public method to calculate the Fermi level.
        *   `void density(double *rho, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double Efmin, double Efmax, int NE, double spin_factor, double Temp, double A)`: Calculates charge density over a range of Fermi levels.
        *   `void derivate(double *drho_dEf, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double Efmin, double Efmax, int NE, double spin_factor, double Temp, double A)`: Calculates the derivative of charge density with respect to the Fermi level.
    *   **Private Members (Variables)**:
        *   `double Eps0`: Vacuum permittivity.
        *   `double e`: Elementary charge.
        *   `double kB`: Boltzmann constant.
    *   **Private Members (Functions)**:
        *   `double Newton(...)`: Implements the Newton-Raphson method to find the Fermi level.
        *   `double get_edge(...)`: Estimates the band edge energy.

## Important Variables/Constants
*   *None defined globally in this file. Physical constants `Eps0`, `e`, `kB` are private members of the `Fermi` class.*

## Usage Examples

```c++
// Example for Fermi class (conceptual)
// Fermi fermi_calculator;
// double calculated_Ef = fermi_calculator.find_fermi(
//     doping_value, charge_sign_indicator, energy_states_array,
//     k_step, num_k_points, k_y_steps, num_k_y_points, k_z_steps, num_k_z_points,
//     num_modes_in_dos, spin_deg_factor, temperature_val, area_val);
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include "Utilities.H"`
*   `#include "Types.H"`

### External Libraries
*   **MPI**: Included via `mpi.h`. While the `Fermi` class itself might not perform extensive MPI communication, its context within the larger OMEN project might involve MPI.

### Interactions with Other Project Components
*   This header file declares the `Fermi` class, which is implemented in `Fermi.C`.
*   The `Fermi` class likely interacts with modules that provide density of states information (e.g., from band structure calculations like the `Singularities` class) and is used by higher-level modules that require self-consistent Fermi level determination (e.g., `EnergyVector` or SCF loops).
*   It depends on `Utilities.H` and `Types.H` for fundamental data types (like `CPX`, though not directly in public signatures here) and possibly utility functions.

---
*This documentation is auto-generated and may require further manual refinement.*
