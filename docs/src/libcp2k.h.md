# Documentation for `src/libcp2k.h`

## Overview

This header file defines the C Application Programming Interface (API) for the CP2K library. It provides a set of `extern "C"` functions that allow external C or C++ programs, such as OMEN, to link against and interact with the CP2K quantum chemistry and solid-state physics software package.

The API allows for:
*   Initializing and finalizing the CP2K environment, with options for managing MPI.
*   Creating and managing "force environments," which represent specific CP2K simulation instances configured by an input file.
*   Setting particle positions and velocities.
*   Calculating energies and forces.
*   Retrieving various results from CP2K.
*   Running a full CP2K input file.
*   Interfacing for quantum transport calculations by defining data structures for parameters (`cp2k_transport_parameters`) and sparse matrices (`cp2k_csr_interop_type`), and allowing registration of a callback function (`ext_method_callback_f_ptr`) for external density matrix evaluation (likely where OMEN hooks in its transport solvers).

The Doxygen comment `/** \brief Definitions for the functions exported in libcp2k.F \author Mohammad Hossein Bani-Hashemian */` indicates its origin and purpose.

## Key Components

### Typedefs
*   `typedef int force_env_t;`: An integer type used as a handle for a CP2K force environment.
*   `typedef struct { ... } cp2k_transport_parameters;`:
    *   **Description**: A C-style struct to pass various parameters relevant to quantum transport calculations between CP2K and the calling library (OMEN). Fields include number of occupied states, number of atoms, energy difference, various method flags, solver choices, thresholds, temperature, etc.
*   `typedef struct { ... } cp2k_csr_interop_type;`:
    *   **Description**: A C-style struct for exchanging sparse matrices in CSR (Compressed Sparse Row) format between CP2K and the calling library. It includes matrix dimensions, non-zero counts, and pointers to the local data arrays (row pointers, column indices, non-zero values).
*   `typedef void (*ext_method_callback_f_ptr) (...);`:
    *   **Description**: A function pointer type for a callback routine. This allows CP2K to call an external function (provided by OMEN) to evaluate the density matrix, passing S and H matrices and expecting P and PImag matrices in return.

### Functions
*(All functions are declared `extern "C"`)*
*   `void cp2k_get_version(char* version_str, int str_length)`: Gets the CP2K version string.
*   `void cp2k_init()`: Initializes CP2K and MPI.
*   `void cp2k_init_without_mpi()`: Initializes CP2K without managing MPI (assuming MPI is already initialized externally).
*   `void cp2k_finalize()`: Finalizes CP2K and MPI.
*   `void cp2k_finalize_without_mpi()`: Finalizes CP2K without touching MPI.
*   `void cp2k_create_force_env(force_env_t* new_force_env, const char* input_file_path, const char* output_file_path)`: Creates a new CP2K simulation instance (force environment) from an input file.
*   `void cp2k_create_force_env_comm(force_env_t* new_force_env, const char* input_file_path, const char* output_file_path, int mpi_comm)`: Creates a force environment with a custom MPI communicator.
*   `void cp2k_destroy_force_env(force_env_t force_env)`: Destroys a previously created force environment.
*   `void cp2k_set_positions(force_env_t force_env, const double* new_pos, int n_el)`: Sets atomic positions.
*   `void cp2k_set_velocities(force_env_t force_env, const double* new_vel, int n_el)`: Sets atomic velocities.
*   `void cp2k_get_results(force_env_t force_env, const char* description, double* result, int n_el)`: Retrieves arbitrary results from CP2K based on a string descriptor.
*   `void cp2k_get_natom(force_env_t force_env, int* natom)`: Gets the number of atoms.
*   `void cp2k_get_nparticle(force_env_t force_env, int* nparticle)`: Gets the number of particles.
*   `void cp2k_get_positions(force_env_t force_env, double* pos, int n_el)`: Gets atomic positions.
*   `void cp2k_get_forces(force_env_t force_env, double* force, int n_el)`: Gets atomic forces.
*   `void cp2k_get_potential_energy(force_env_t force_env, double* e_pot)`: Gets the potential energy.
*   `void cp2k_calc_energy_force(force_env_t force_env)`: Triggers a calculation of energy and forces.
*   `void cp2k_calc_energy(force_env_t force_env)`: Triggers a calculation of energy only.
*   `void cp2k_run_input(const char* input_file_path, const char* output_file_path)`: Runs a full CP2K simulation based on an input file.
*   `void cp2k_run_input_comm(const char* input_file_path, const char* output_file_path, int mpi_comm)`: Runs a CP2K input with a custom MPI communicator.
*   `void cp2k_transport_set_callback(force_env_t force_env, ext_method_callback_f_ptr func)`: Registers the external callback function (like OMEN's `c_scf_method`) to be used by CP2K for density matrix calculations in transport simulations.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual: Initializing CP2K and creating a force environment
// force_env_t my_env;
// cp2k_init();
// cp2k_create_force_env(&my_env, "input.cp2k", "output.cp2k");

// Conceptual: Setting a callback for transport calculations
// cp2k_transport_set_callback(my_env, &my_omen_density_calculator_function);
// cp2k_calc_energy_force(my_env); // This would trigger OMEN's function via callback

// Conceptual: Finalizing
// cp2k_destroy_force_env(my_env);
// cp2k_finalize();
```

## Dependencies and Interactions

### Included Files
*   `#include <stdbool.h>`

### External Libraries
*   **CP2K**: This header file *is* the C interface to the CP2K library. The actual CP2K library (Fortran-based) must be linked against.
*   **MPI**: MPI is intrinsically linked with CP2K's parallel operation. Several functions explicitly deal with MPI communicators or manage MPI initialization/finalization.

### Interactions with Other Project Components
*   This file is the primary bridge if OMEN is using CP2K as its DFT engine.
*   OMEN's `c_scf.C` (`c_scf_method`) is designed to be the callback function registered via `cp2k_transport_set_callback`.
*   OMEN would use the `cp2k_csr_interop_type` to receive Hamiltonian (H) and Overlap (S) matrices from CP2K and to send back the Density (P) matrix.
*   The `cp2k_transport_parameters` struct is used to pass settings from CP2K's input to OMEN's transport module.

---
*This documentation is auto-generated and may require further manual refinement.*
