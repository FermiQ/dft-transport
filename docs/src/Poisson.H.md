# Documentation for `src/Poisson.H`

## Overview

This header file defines the `Poisson` class, which is responsible for solving the Poisson equation to determine the electrostatic potential distribution within the simulated device. This is a fundamental part of self-consistent electronic structure and transport calculations, as the potential influences the charge distribution, and vice-versa. The class supports both 1D Finite Difference Method (FDM) and 3D Finite Element Method (FEM) discretizations.

## Key Components

### Classes/Structs
*   `class Poisson`:
    *   **Description**: Manages the setup and solution of the Poisson equation.
    *   **Public Members (Variables)**:
        *   `TCSR<double> *P`: Pointer to the FDM Poisson matrix (1D).
        *   `TCSR<double> *P1`: Pointer to a FEM Poisson matrix component (potentially unused or for specific parts).
        *   `TCSR<double> *P2`: Pointer to the full FEM Poisson matrix.
        *   `TCSR<double> *RP2`: Pointer to the reduced FEM Poisson matrix (after considering only active Poisson nodes).
    *   **Public Members (Functions)**:
        *   `Poisson()`: Constructor.
        *   `~Poisson()`: Destructor.
        *   `void init(WireGenerator* Wire, WireStructure* nanowire, FEMGrid* FEM, int max_proc_poisson, int poisson_solver, MPI_Comm vg_comm)`: Initializes the Poisson solver, including creating the discretization matrix (FDM or FEM).
        *   `void solve(double* Vnew, double* Vold, double* rho_atom, double* drho_atom_dV, double sign_rho, FermiStructure* FS, FEMGrid* FEM, WireGenerator* Wire, double Temp, double* Vg, double Vground, double Vsource, double Vdrain, double* residual, double poisson_criterion, int poisson_iteration, int max_proc_poisson, int poisson_solver, MPI_Comm loc_comm, MPI_Comm glob_comm, int bcast_size, MPI_Comm bcast_comm, int no_task)`: The main iterative solver routine for the Poisson equation.
    *   **Private Members (Variables)**:
        *   `int N3D`: Constant for 3 dimensions.
        *   `int mpi_rank, mpi_size`: MPI rank and size of the default communicator.
        *   `int findx`: Fortran-style indexing flag.
        *   `int del_PMatrix, del_P1DMatrix`: Flags to control matrix deallocation in the destructor.
        *   `double Eps0, e, kB`: Physical constants (vacuum permittivity, elementary charge, Boltzmann constant).
        *   `double Vol`: Atomic volume (from `WireGenerator`).
    *   **Private Members (Functions - a selection)**:
        *   `void solve_FDM(...)`: Implements the FDM Poisson solver (1D).
        *   `void solve_FEM(...)`: Implements the FEM Poisson solver (3D).
        *   `void create_FEM_matrix(...)`: Assembles the FEM stiffness matrix.
        *   `void create_FDM_matrix(...)`: Assembles the FDM discretization matrix.
        *   `void reduce_P(...)`: Reduces the FEM matrix to active nodes.
        *   `void init_V(...)`: Initializes potential with boundary conditions.
        *   `void iterate_V(...)`: Updates potential in the iterative solver.
        *   `void find_fermi(...)`, `void find_1D_fermi(...)`: Calculate quasi-Fermi levels.
        *   `void init_doping(...)`: Prepares doping vector for the solver.
        *   `void calc_cd_cdd(...)`, `void calc_1D_cd_cdd(...)`: Calculate charge density and its derivative.
        *   `void geometry(...)`, `double get_permitivity(...)`, `void get_points(...)`: Geometric helpers for FEM matrix assembly.

## Important Variables/Constants
*   *None defined globally in this file. Physical constants are private members of the class.*

## Usage Examples

```c++
// Conceptual example for Poisson class usage
// WireGenerator wire_gen_obj;
// WireStructure device_struct_obj;
// FEMGrid fem_grid_obj;
// // ... initialize wire_gen_obj, device_struct_obj, fem_grid_obj ...

// Poisson poisson_solver_obj;
// MPI_Comm current_comm = MPI_COMM_WORLD;
// poisson_solver_obj.init(&wire_gen_obj, &device_struct_obj, &fem_grid_obj,
//                         num_procs_poisson, solver_type_flag, current_comm);

// double potential_solution[num_grid_nodes];
// double potential_initial_guess[num_grid_nodes];
// double charge_density_atoms[num_atoms];
// // ... and other parameters for solve() ...
// poisson_solver_obj.solve(potential_solution, potential_initial_guess, charge_density_atoms, ...);
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include "WireGenerator.H"`
*   `#include "Utilities.H"`
*   `#include "Blas.H"`
*   `#include "Types.H"`
*   `#include "FEMGrid.H"`
*   `#include "CSR.H"`
*   `#include "Hypre.H"`
*   `#include "Fermi.H"`

### External Libraries
*   **MPI**: Used for parallel processing.
*   **Hypre**: Used as a linear solver backend (specifically `Hypre<double>`) for the FEM Poisson equation.
*   **BLAS**: Implied by `Blas.H` for low-level vector operations.

### Interactions with Other Project Components
*   This header file declares the `Poisson` class, which is implemented in `Poisson.C`.
*   The `Poisson` class is a core component for self-consistent simulations, calculating the electrostatic potential that influences charge distribution.
*   It interacts heavily with:
    *   `WireGenerator.H`: To get atomic volume and device dimensionality.
    *   `FEMGrid.H`: To get the discretized grid, tetrahedral mesh, material permittivity, doping profiles, and contact region indices.
    *   `Types.H`: For structures like `WireStructure`, `FermiStructure`, `GATE`, `DOPING`.
    *   `CSR.H`: For `TCSR<double>` to represent the sparse Poisson matrix.
    *   `LinearSolver.H` and `Hypre.H`: Uses the Hypre solver.
    *   `Fermi.H`: Uses the `Fermi` class for calculating quasi-Fermi levels in 1D.
*   The calculated potential is used by other modules (e.g., `Density.C` or the main SCF loop) to update the Hamiltonian.

---
*This documentation is auto-generated and may require further manual refinement.*
