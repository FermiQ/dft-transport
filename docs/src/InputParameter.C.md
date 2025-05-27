# Documentation for `src/InputParameter.C`

## Overview

This file is responsible for the management of global simulation parameters within the OMEN project. It provides functions to:
1.  Initialize global parameter structures (`PARAM`, `WireStructure`, `ENERGY`, `VOLTAGE`, and nested structures like `Strain`, `Roughness`, `Alloy`) by allocating memory and setting them to default values (`init_parameters`, `default_parameters`).
2.  Clean up and deallocate the memory used by these global parameter structures (`delete_parameters`).
3.  Offer a C-style interface through numerous `extern "C" void init_...(...)` functions. These functions allow individual parameters within the global structures to be set, which is likely used for parsing input files or for interfacing with C code.

The parameters managed here control a wide range of simulation aspects, including device geometry, material properties, solver settings, bias conditions, and output options.

## Key Components

### Functions
*   `void init_parameters()`:
    *   **Description**: Allocates memory for the primary global parameter structures (`parameter`, `nanowire`, `En`, `voltage`, and their nested structures like `nanowire->strain`, `nanowire->rough`, etc.). It then calls `default_parameters()` to set initial values.
*   `void delete_parameters()`:
    *   **Description**: Deallocates all memory previously allocated by `init_parameters` for the global parameter structures and their members, preventing memory leaks.
*   `void default_parameters()`:
    *   **Description**: Sets default values for all fields within the global parameter structures (`nanowire`, `En`, `parameter`, `voltage`). This ensures that all simulation parameters have a defined state before being potentially overridden by user inputs.
*   `extern "C" void init_...(...)` Functions:
    *   **Description**: A large suite of C-callable functions designed to set individual parameters within the global structures. Each function typically takes one or more arguments corresponding to the parameter(s) it modifies. Examples include:
        *   `init_restart(int restart, int IG_start, int IS_start, int ID_start)`
        *   `init_tb(int tb)`
        *   `init_a0_c0_u0(double a0, double c0, double u0)`
        *   `init_mat_name(char *mat_name)`
        *   `init_ndim(int NDim)`
        *   `init_strain(int on)`
        *   `init_Eps_xx(double Eps_xx)`
        *   `init_no_mat(int no_mat)`
        *   `init_mat_type(int mat_index, char* type)`
        *   `init_nvg(int NVG, int no_diff_gate)`
        *   `init_vgmin(double Vgmin1, double Vgmin2, double Vgmin3, double Vgmin4)`
        *   `init_lattype(const char *latname)`
    *   *(This is not an exhaustive list; many such functions exist for almost every parameter field in the global structures.)*

### Classes/Structs
*   *None defined in this file. It operates on global instances of structs defined in `Types.H`.*

## Important Variables/Constants
*   *(External global pointers, defined elsewhere but heavily used and modified here)*
    *   `extern PARAM *parameter;`
    *   `extern WireStructure *nanowire;`
    *   `extern ENERGY *En;`
    *   `extern VOLTAGE *voltage;`
    *   `extern Strain *strain;` (Note: `nanowire->strain` is used, so this global `strain` might be redundant or for a different purpose if not a typo in original code's extern context).

## Usage Examples

```c++
// Typically, init_parameters() is called at the beginning of the program.
// init_parameters();

// Then, parser functions would call the various init_... functions:
// init_ndim(2); // Example: Set simulation to 2D
// init_a0_c0_u0(0.565, 0.565, 0.0); // Set lattice constants

// At the end of the program, delete_parameters() is called.
// delete_parameters();
```

## Dependencies and Interactions

### Included Files
*   `#include <iostream>`
*   `#include <string.h>`
*   `#include "InputParameter.H"`
*   `#include "Types.H"`

### External Libraries
*   *None directly identifiable from this file's content.*

### Interactions with Other Project Components
*   This file implements functions declared in `InputParameter.H`.
*   It is critically linked to `Types.H`, as it allocates, initializes, and deallocates instances of the structures defined there (e.g., `PARAM`, `WireStructure`, `ENERGY`, `VOLTAGE`).
*   The global parameter variables modified by this file are used by virtually all other modules in the OMEN project to control their behavior and access device/simulation settings.
*   The C-style `init_...` functions provide an API for setting up the simulation, likely used by an input file parser.

---
*This documentation is auto-generated and may require further manual refinement.*
