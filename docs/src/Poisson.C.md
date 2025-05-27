# Documentation for `src/Poisson.C`

## Overview

This file implements the `Poisson` class, which is responsible for solving the Poisson equation to determine the electrostatic potential within the simulated device. This is a critical step in self-consistent simulations where the charge distribution affects the potential, and the potential, in turn, affects the charge distribution.

The class supports:
*   Both 1D (Finite Difference Method - FDM) and 3D (Finite Element Method - FEM) Poisson solves.
*   Initialization of the Poisson matrix based on the device geometry, FEM grid, and material permittivities.
*   An iterative Newton-Raphson like solver loop that updates the potential based on the current charge distribution and its derivative with respect to potential.
*   Application of boundary conditions for gates, source, drain, and ground contacts.
*   Calculation of charge densities and quasi-Fermi levels as part of the iterative solution.

## Key Components

### Functions
*(These are member functions of the `Poisson` class, declared in `Poisson.H`)*
*   `Poisson()`: Constructor. Initializes physical constants (`Eps0`, `e`, `kB`), MPI rank/size, and default flags.
*   `~Poisson()`: Destructor. Frees memory allocated for Poisson matrices.
*   `void init(WireGenerator *Wire, WireStructure *nanowire, FEMGrid *FEM, int max_proc_poisson, int poisson_solver, MPI_Comm vg_comm)`:
    *   **Description**: Initializes the Poisson solver. For 1D, it calls `create_FDM_matrix`. For 3D, it calls `create_FEM_matrix` and `reduce_P`.
*   `void solve(double *Vnew, double *Vold, double *rho_atom, double *drho_atom_dV, double sign_rho, FermiStructure *FS, FEMGrid* FEM, WireGenerator *Wire, double Temp, double *Vg, double Vground, double Vsource, double Vdrain, double *residual, double poisson_criterion, int poisson_iteration, int max_proc_poisson, int poisson_solver, MPI_Comm loc_comm, MPI_Comm glob_comm, int bcast_size, MPI_Comm bcast_comm, int no_task)`:
    *   **Description**: The main public method to solve the Poisson equation iteratively. It calls either `solve_FDM` or `solve_FEM` based on the device dimensionality.
*   `void solve_FDM(...)`: Implements the iterative FDM Poisson solver for 1D systems.
*   `void solve_FEM(...)`: Implements the iterative FEM Poisson solver for 3D systems. Uses a `Hypre<double>` linear solver.
*   `void init_V(double *Vnew, double *Vold, int NGrid, int **gate_index, int *NGate, int no_diff_gate, double *Vg, int *ground_index, int NGround, double Vground, int *source_index, int NSource, double Vsource, int *drain_index, int NDrain, double Vdrain)`:
    *   **Description**: Initializes the potential array `Vnew` by copying `Vold` and then applying fixed potentials for gates, ground, source, and drain contacts based on their grid indices.
*   `void iterate_V(double *Vnew, double *delta_V, int *poisson_index, int NGrid)`:
    *   **Description**: Updates the potential `Vnew` by subtracting `delta_V` (the solution of the linearized Poisson equation). If `poisson_index` is provided, only updates nodes active in the Poisson solve.
*   `void find_fermi(double *Ef, double *rhsign, double *Vnew, double *rho_atom, double NC, int *real_at_index, int NAtom, double Temp)`:
    *   **Description**: (Likely for 3D FEM) Calculates quasi-Fermi levels `Ef` for each atom based on the current potential `Vnew` and atomic charge densities `rho_atom`.
*   `void find_1D_fermi(double *Ef, double *rhsign, double *Vnew, double *rho, double *drho_dV, Fermi *FF, FermiStructure *FS, double *rho_atom, double *drho_atom_dV, int NAtom, double NC, double *grid, int start, int stop, double Temp)`:
    *   **Description**: Calculates quasi-Fermi levels for 1D FDM grid points, potentially using a `Fermi` object if detailed band structure is involved.
*   `void init_doping(double *doping, double *full_doping, int *poisson_index, int NGrid, int start, int stop)`:
    *   **Description**: Populates the local `doping` array for the current MPI process based on the `full_doping` array and the `poisson_index` map.
*   `void copy_cd_cdd(double *rho, double *drho_dV, double *rho_atom, double *drho_atom_dV, int *ratom_index, int *ch_conv, int *poisson_index, int *atom_index, int NAtom, int start, int stop)`:
    *   **Description**: Copies charge density (`rho_atom`) and its derivative (`drho_atom_dV`) to local arrays `rho` and `drho_dV` for active Poisson nodes.
*   `void calc_cd_cdd(double *rho, double *drho_dV, double *rhsign, double *Ef, double *Vnew, double NC, int *real_at_index, int *ratom_index, int *ch_conv, int *poisson_index, int *atom_index, int NAtom, double Temp, int start, int stop)`:
    *   **Description**: Calculates charge density `rho` and its derivative `drho_dV` at FEM nodes based on the current potential `Vnew` and quasi-Fermi levels `Ef`, using a Boltzmann approximation.
*   `void calc_1D_cd_cdd(double *rho, double *drho_dV, double *rhsign, double *Ef, double *Vnew, Fermi *FF, FermiStructure *FS, double NC, double Temp, int start, int stop)`:
    *   **Description**: 1D version of `calc_cd_cdd`, potentially using a `Fermi` object for more accurate DOS integration.
*   `void create_FEM_matrix(int *poisson_index, int NPoisson, int *tetrahedron, double *tetraVol, int NTetra, double *grid, int NGrid, double *Eps)`:
    *   **Description**: Assembles the FEM stiffness matrix (`P2`) for the Poisson equation based on the tetrahedral mesh, grid point coordinates, and permittivity values.
*   `void assemble_matrix(int no_element, double *mp1, double *mp2, IJPOS *ij, int *poisson_index, int NPoisson, int NGrid)`:
    *   **Description**: Helper function for `create_FEM_matrix`. Takes elemental matrix contributions and assembles them into the global sparse matrix `P2`. `mp1` seems unused.
*   `void create_FDM_matrix(FEMGrid *FEM)`:
    *   **Description**: Assembles the FDM matrix (`P`) for a 1D Poisson problem.
*   `void reduce_P(int *poisson_index, int NGrid)`:
    *   **Description**: Reduces the full FEM matrix `P2` to `RP2` by keeping only rows and columns corresponding to nodes active in the Poisson solve (where `poisson_index` is true).
*   `void init_j_shift(int *j_shift, int *poisson_index, int NGrid)`: Helper for `reduce_P` to create column index shifts.
*   `double get_permitivity(double *Eps, int *tetrahedron)`: Calculates the average permittivity for a tetrahedron.
*   `void get_points(double *p, int *tetrahedron, double *grid, int i_pos)`: Helper for FEM matrix assembly; gets coordinates of 3 vertices of a tetrahedron face.
*   `void geometry(double *A, double *h, double *p1, double *p2, double *p3, double *p4)`: Helper for FEM matrix assembly; calculates geometric factors (face normal area vectors `A`, heights `h`) for a tetrahedron.
*   `void sort_ij(IJPOS* ij, int *index, int *IMIN, int *IMAX, int no_element)`: Sorts elemental matrix entries by row and then column index.
*   `void generate_output(const char *filename, double *data, int NR, int NC)`: Writes a dense double matrix/vector to file.
*   `void generate_output(const char *filename, int *data, int NR, int NC)`: Writes a dense integer matrix/vector to file.

### Classes/Structs
*   *None defined in this file. It implements methods for the `Poisson` class declared in `Poisson.H`.*

## Important Variables/Constants
*   *(Member variables of the `Poisson` class, initialized in the constructor)*
    *   `N3D`: Constant for 3 dimensions (value 3).
    *   `Eps0`: Vacuum permittivity (8.854e-12 F/m).
    *   `e`: Elementary charge (1.6022e-19 C).
    *   `kB`: Boltzmann constant (1.38e-23 J/K).
    *   `findx`: Fortran-style indexing flag (value 1).
    *   `del_PMatrix`, `del_P1DMatrix`: Flags to control destructor behavior.

## Usage Examples

```c++
// Poisson solver is typically used within a self-consistent loop (e.g., NEGF-Poisson).
// WireGenerator wire;
// WireStructure params;
// FEMGrid fem_grid;
// // ... initialize wire, params, fem_grid ...

// Poisson poisson_solver;
// MPI_Comm comm = MPI_COMM_WORLD;
// poisson_solver.init(&wire, &params, &fem_grid, num_procs, 1, comm);

// double potential_new[NGrid_nodes], potential_old[NGrid_nodes];
// double charge_density[NAtoms], charge_deriv[NAtoms];
// double residual_error;
// // ... initialize potential_old, charge_density, etc. ...
// poisson_solver.solve(potential_new, potential_old, charge_density, charge_deriv,
//                      ... other_params ..., &residual_error, ...);
```

## Dependencies and Interactions

### Included Files
*   `#include "Poisson.H"`
*   (Implicitly, via `Poisson.H`): `<mpi.h>`, `"WireGenerator.H"`, `"Utilities.H"`, `"Blas.H"`, `"Types.H"`, `"FEMGrid.H"`, `"CSR.H"`, `"LinearSolver.H"`, `"Hypre.H"`.

### External Libraries
*   **MPI**: Used for parallel communication and distributed solving.
*   **Hypre**: Used as the underlying linear solver for the FEM Poisson equation (`Hypre<double>`).
*   **BLAS**: Used for vector operations like `c_dcopy`, `c_daxpy`, `max_sign_abs_vec` (via `Blas.H`).

### Interactions with Other Project Components
*   This file implements the `Poisson` class declared in `Poisson.H`.
*   **`WireGenerator.H` & `FEMGrid.H`**: Uses `WireGenerator` and `FEMGrid` objects to get device geometry, mesh, material properties (permittivity), and doping information.
*   **`Types.H`**: Uses various structures like `WireStructure`, `GATE`, `DOPING`, `MAT`, `FermiStructure`, `XYZPOS`, `IJPOS`.
*   **`CSR.H`**: Uses `TCSR<double>` for representing the discretized Poisson matrix.
*   **`LinearSolver.H` & `Hypre.H`**: Uses the `Hypre<double>` solver, which is an implementation of the `LinearSolver<double>` interface.
*   **`Fermi.H`**: Uses the `Fermi` class to calculate quasi-Fermi levels in 1D.
*   The calculated potential is a crucial input for updating the Hamiltonian in self-consistent simulations.

---
*This documentation is auto-generated and may require further manual refinement.*
