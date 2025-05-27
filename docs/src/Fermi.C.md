# Documentation for `src/Fermi.C`

## Overview

This file implements the `Fermi` class. The primary responsibility of this class is to calculate the Fermi level (chemical potential) in a semiconductor material. It achieves this by using the material's density of states (DOS) – provided as energy eigenvalues `Ek` and k-space discretization information (`dk`, `dky`, `dkz`, etc.) – and the net doping concentration `Ndop`. The core of the calculation involves numerically solving the charge neutrality equation, typically employing methods like the Newton-Raphson iteration to find the Fermi level that yields the specified doping concentration.

## Key Components

### Functions
*(These are member functions of the `Fermi` class, declared in `Fermi.H`)*
*   `Fermi()`:
    *   **Description**: Constructor. Initializes fundamental physical constants like `Eps0` (vacuum permittivity), `e` (elementary charge), and `kB` (Boltzmann constant).
*   `~Fermi()`:
    *   **Description**: Destructor. Currently empty.
*   `double find_fermi(double Ndop, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double spin_factor, double Temp, double A)`:
    *   **Description**: The main public method to find the Fermi level. It first estimates a range for the Fermi level using `get_edge`, then refines this using a preliminary scan with the `density` function, and finally uses the `Newton` method for a precise value.
    *   `sign_rho`: Indicates whether to calculate electron (typically +1) or hole (typically -1) density relative to the band edge.
    *   `A`: Cross-sectional area.
*   `double Newton(double Ef0, double Ndop, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double spin_factor, double Temp, double A)`:
    *   **Description**: Implements the Newton-Raphson iterative method to find the Fermi level `Ef` such that the calculated charge density `n(Ef)` matches the target doping `Ndop`. It uses the `density` and `derivate` functions.
*   `double get_edge(double *Ek, double sign, int Nkyz, int Nk, int NM)`:
    *   **Description**: Determines an initial estimate for the band edge (minimum or maximum energy relevant for the calculation) based on the sign of the charge carriers.
*   `void density(double *rho, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double Efmin, double Efmax, int NE, double spin_factor, double Temp, double A)`:
    *   **Description**: Calculates the charge density `rho` for a sweep of `NE` Fermi levels between `Efmin` and `Efmax`. It integrates the Fermi-Dirac distribution multiplied by the density of states.
*   `void derivate(double *drho_dEf, double sign_rho, double *Ek, double dk, int Nk, double *dky, int Nky, double *dkz, int Nkz, int n_of_modes, double Efmin, double Efmax, int NE, double spin_factor, double Temp, double A)`:
    *   **Description**: Calculates the derivative of the charge density with respect to the Fermi level, `d(rho)/d(Ef)`, for a sweep of Fermi levels. This is used in the Newton-Raphson method.

### Classes/Structs
*   *None defined in this file. It implements methods for the `Fermi` class declared in `Fermi.H`.*

## Important Variables/Constants
*   *(Member variables of the `Fermi` class, initialized in the constructor)*
    *   `Eps0`: Vacuum permittivity (8.854e-12 F/m).
    *   `e`: Elementary charge (1.6022e-19 C).
    *   `kB`: Boltzmann constant (1.38e-23 J/K).

## Usage Examples

```c++
// Conceptual example of using the Fermi class
// Fermi fermi_calculator;
// double doping_concentration = 1e18; // example doping
// double charge_sign = 1.0; // for electrons
// // Assume Ek_data, dk_val, Nk_val, dky_vals, Nky_val, dkz_vals, Nkz_val,
// // num_modes, spin, temperature, area are all initialized.
// double calculated_fermi_level = fermi_calculator.find_fermi(
//     doping_concentration, charge_sign, Ek_data, dk_val, Nk_val,
//     dky_vals, Nky_val, dkz_vals, Nkz_val, num_modes,
//     spin, temperature, area);
```

## Dependencies and Interactions

### Included Files
*   `#include "Fermi.H"`

### External Libraries
*   *None directly identifiable from includes in this file, relies on standard C++ math functions.*

### Interactions with Other Project Components
*   This file implements the methods of the `Fermi` class, which is declared in `Fermi.H`.
*   The `Fermi` class is likely used by other modules that require self-consistent determination of Fermi levels, such as the `Singularities` class (which determines band edges and might call this to align Fermi levels to doping) or the `EnergyVector` class (which sets up energy integration meshes based on Fermi levels).
*   It uses the `fermi` function (from `CSR.H` or `Utilities.H`) for the Fermi-Dirac distribution.
*   It implicitly depends on `Types.H` if `CPX` or other custom types were used (though this specific file primarily uses `double`).

---
*This documentation is auto-generated and may require further manual refinement.*
