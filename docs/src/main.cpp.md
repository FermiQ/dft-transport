# Documentation for `src/main.cpp`

## Overview

This file contains the `main` function, which serves as the primary entry point for the OMEN (Open-source MEsoscopic Nanostructure) simulation program when it is coupled with the CP2K quantum chemistry package. The program initializes the MPI environment, then initializes CP2K for QM/MM calculations. It registers OMEN's `c_scf_method` as a callback function that CP2K will call to perform quantum transport calculations (likely for determining the density matrix based on CP2K's Hamiltonian).

Optionally, if OMEN's internal Poisson solver is compiled (indicated by `HAVE_OMEN_POISSON`), this `main` function also handles the parsing of a second input file (presumably OMEN's specific input for geometry and simulation parameters), initializes OMEN's internal structures like `WireGenerator` and `FEMGrid`, and sets up the Poisson solver.

The typical workflow involves:
1.  MPI Initialization.
2.  (Conditional) OMEN-specific setup: Parse OMEN input, generate wire geometry, create FEM grid, initialize Poisson solver.
3.  CP2K Initialization: Initialize CP2K environment, create a "force environment" from a CP2K input file.
4.  Callback Registration: Register `c_scf_method` with CP2K for transport calculations.
5.  Execution: Trigger CP2K to calculate energy and forces. During this process, CP2K will call back into OMEN's `c_scf_method` when needed.
6.  Result Retrieval: Get potential energy, forces, positions from CP2K (example calls shown).
7.  Finalization: Destroy CP2K force environment, finalize CP2K, (conditional) delete OMEN objects, and finalize MPI.

## Key Components

### Functions
*   `int main(int argc, char **argv)`:
    *   **Description**: The main entry point of the OMEN application. It orchestrates the initialization, execution, and finalization of both OMEN-specific components (if enabled) and the CP2K library.
    *   **Parameters**:
        *   `argc`: Argument count.
        *   `argv`: Argument vector. `argv[1]` is expected to be the CP2K input file. If `HAVE_OMEN_POISSON` is defined and `argc > 2`, `argv[2]` is expected to be the OMEN input file.
*   `void c_scf_method(cp2k_transport_parameters cp2k_transport_params, cp2k_csr_interop_type S, cp2k_csr_interop_type KS, cp2k_csr_interop_type* P, cp2k_csr_interop_type* PImag);`
    *   **Description**: This is a forward declaration of the callback function that OMEN provides to CP2K. Its implementation is in `c_scf.C`. CP2K calls this function to have OMEN compute the density matrix (`P`, `PImag`) given the overlap (`S`) and Kohn-Sham (`KS`) matrices.

### Extern "C" Functions
*(Conditionally compiled if `HAVE_OMEN_POISSON` is defined)*
*   `void yyrestart(FILE *);`: Function from Flex (lexical analyzer) to restart the parser with a new input file.
*   `void yyparse();`: Function from Bison (parser generator) to parse the input file.
*   `extern FILE *yyin;`: Global variable used by Flex/Bison to read from the input file.

## Important Variables/Constants
*(Global pointers, conditionally compiled if `HAVE_OMEN_POISSON` is defined. These are typically defined in `InputParameter.C` or a similar global scope file and declared as `extern` here for access within `main`.)*
*   `PARAM *parameter;`
*   `WireStructure *nanowire;`
*   `ENERGY *En;`
*   `VOLTAGE *voltage;`
*   `WireGenerator* Wire;`
*   `FEMGrid *FEM;`
*   `Poisson *OMEN_Poisson_Solver;`

## Usage Examples

```bash
# Example command line execution:
# mpirun -np <num_procs> omen_cp2k_executable cp2k_input.inp omen_input.inp
# where:
#   cp2k_input.inp: Input file for CP2K settings.
#   omen_input.inp: (Optional, if HAVE_OMEN_POISSON) Input file for OMEN specific settings.
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <vector>`
*   `#include <assert.h>`
*   `#include "libcp2k.h"`
*   Conditional includes (if `HAVE_OMEN_POISSON` is defined):
    *   `#include "Types.H"`
    *   `#include "InputParameter.H"`
    *   `#include "WireGenerator.H"`
    *   `#include "FEMGrid.H"`
    *   `#include "Poisson.H"`

### External Libraries
*   **MPI**: Used for parallel processing.
*   **CP2K**: The core quantum mechanical engine OMEN interfaces with, via `libcp2k.h`.
*   **(Conditional) Flex/Bison**: If `HAVE_OMEN_POISSON` is defined, `yyparse` and `yyrestart` suggest the use of Flex and Bison for parsing OMEN's specific input file.

### Interactions with Other Project Components
*   This file is the main driver of the OMEN application when coupled with CP2K.
*   **`libcp2k.h`**: Uses functions from this C API to control CP2K (e.g., `cp2k_init_without_mpi`, `cp2k_create_force_env_comm`, `cp2k_transport_set_callback`, `cp2k_calc_energy_force`).
*   **`c_scf.C`**: The `c_scf_method` function, implemented in `c_scf.C`, is passed as a callback to CP2K.
*   **(Conditional) `InputParameter.H` / `.C`**: If OMEN's Poisson solver is used, `init_parameters()` and `delete_parameters()` are called to manage global simulation parameters.
*   **(Conditional) `Types.H`**: If OMEN's Poisson solver is used, this file uses global variables whose types are defined in `Types.H` (e.g., `PARAM`, `WireStructure`).
*   **(Conditional) `WireGenerator.H` / `.C`**: If OMEN's Poisson solver is used, an instance of `WireGenerator` is created and used to define the device geometry.
*   **(Conditional) `FEMGrid.H` / `.C`**: If OMEN's Poisson solver is used, an instance of `FEMGrid` is created and used to generate the FEM mesh.
*   **(Conditional) `Poisson.H` / `.C`**: If OMEN's Poisson solver is used, an instance of `Poisson` is created to solve the Poisson equation.

---
*This documentation is auto-generated and may require further manual refinement.*
