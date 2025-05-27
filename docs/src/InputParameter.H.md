# Documentation for `src/InputParameter.H`

## Overview

This header file declares a suite of C-style functions that serve as an Application Programming Interface (API) for initializing and managing global simulation parameters within the OMEN project. These functions are designed to be callable from C code (hence `extern "C"`) and are likely used by an input file parser or a setup module to configure the simulation environment. The declarations cover functions to initialize all parameters to defaults, delete (deallocate) parameters, and set individual parameter values.

## Key Components

### Functions
*(All functions are declared `extern "C"`)*
*   `void init_parameters();`: Initializes (allocates and sets defaults for) all global parameter structures.
*   `void default_parameters();`: Sets all global parameters to their default values.
*   `void delete_parameters();`: Deallocates memory used by global parameter structures.
*   **Individual Parameter Initialization Functions**:
    *   A large number of functions named `init_<parameter_name>(...)` are declared. These functions take arguments corresponding to the values of specific simulation parameters and are used to set these values in the global parameter structures. Examples include:
        *   `void init_tb(int tb_value);`
        *   `void init_dsp3(double dsp3_value);`
        *   `void init_a0_c0_u0(double a0, double c0, double u0);`
        *   `void init_first_atom(char* atom_name);`
        *   `void init_mat_name(char* material_name);`
        *   `void init_ndim(int num_dimensions);`
        *   `void init_Eps_xx(double strain_component);`
        *   `void init_no_mat(int num_materials);`
        *   `void init_mat_type(int material_index, char* type_name);`
        *   `void init_nvg(int num_voltage_gates, int num_different_gates);`
        *   `void init_vgmin(double vgmin1, double vgmin2, double vgmin3, double vgmin4);`
        *   `void init_lattype(const char* lattice_name);`
        *   *(This list is representative, not exhaustive. The header contains declarations for many specific parameters related to device geometry, materials, solvers, bias conditions, output files, etc.)*

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined in this file; it contains only function declarations.*

## Usage Examples

```c++
// Conceptual usage within an input parsing mechanism or main setup

// extern "C" {
// #include "InputParameter.H"
// }

// // At the start of the simulation setup
// init_parameters();

// // While parsing an input file or setting up parameters
// init_ndim(3);
// init_mat_name("Silicon");
// init_a0_c0_u0(0.543, 0.543, 0.0);
// // ... call other init_... functions ...

// // At the end of the simulation
// delete_parameters();
```

## Dependencies and Interactions

### Included Files
*   *The provided snippet does not show includes, but a typical C header might include standard headers if needed for types like `char*`. The corresponding `.C` file includes `"Types.H"`.*

### External Libraries
*   *None directly identifiable from the header content.*

### Interactions with Other Project Components
*   This header file declares functions that are implemented in `InputParameter.C`.
*   These functions provide the primary C-style API for configuring the simulation by modifying global parameter structures. These global structures (like `nanowire`, `parameter`, `En`, `voltage` defined in `Types.H`) are used throughout the OMEN application to control behavior.
*   It forms a crucial link between user input (e.g., from an input file) and the internal state of the simulation.

---
*This documentation is auto-generated and may require further manual refinement.*
