# Documentation for `src/FEMGrid.H`

## Overview

This header file defines the `FEMGrid` class, which is responsible for creating, managing, and providing access to the Finite Element Method (FEM) grid. This grid is primarily used for electrostatic calculations (e.g., solving the Poisson equation) within the OMEN project. The class handles the generation of grid points from an atomic structure, performs Delaunay triangulation to create a mesh, and maps physical properties like doping and permittivity onto this grid.

## Key Components

### Classes/Structs
*   `class FEMGrid`:
    *   **Description**: Manages the FEM grid, including its generation from atomic coordinates, Delaunay triangulation, and attribution of physical properties.
    *   **Public Members (Variables)**:
        *   `double *grid`: Pointer to an array storing the coordinates of the grid points.
        *   `double *tetraVol`: Pointer to an array storing the volumes of the tetrahedra in the mesh.
        *   `double *doping`: Pointer to an array storing doping concentrations at grid points.
        *   `double *Eps`: Pointer to an array storing permittivity values at grid points.
        *   `int *tetrahedron`: Pointer to an array defining the connectivity of the tetrahedral mesh (indices of the four grid points forming each tetrahedron).
        *   `int *atom_index`: Maps original atom indices to grid point indices.
        *   `int *real_at_index`: Maps simulation atom indices to grid point indices.
        *   `int *ratom_index`: Reduced atom index mapping, likely for atoms in the Poisson domain.
        *   `int *channel_index`: Indices of grid points belonging to the channel.
        *   `int *poisson_index`: Flags grid points included in the Poisson solve.
        *   `int **gate_index`: Array of pointers to store indices of grid points belonging to different gates.
        *   `int *ground_index`: Indices of grid points belonging to ground contacts.
        *   `int *source_index`: Indices of grid points belonging to the source contact.
        *   `int *drain_index`: Indices of grid points belonging to the drain contact.
        *   `int NAtom`: Number of atoms considered for grid generation.
        *   `int NGrid`: Total number of grid points.
        *   `int NTetra`: Total number of tetrahedra in the mesh.
        *   `int NChannel`: Number of grid points in the channel.
        *   `int NPoisson`: Number of grid points in the Poisson domain.
        *   `int *NGate`: Array storing the number of grid points for each gate.
        *   `int NGround`: Number of grid points on ground contacts.
        *   `int no_diff_gate`: Number of different gates.
        *   `int NSource`: Number of grid points on the source contact.
        *   `int NDrain`: Number of grid points on the drain contact.
    *   **Public Members (Functions)**:
        *   `FEMGrid()`: Constructor.
        *   `~FEMGrid()`: Destructor.
        *   `void execute_task(WireGenerator*, WireStructure*, int, int, MPI_Comm, MPI_Comm)`: Main function to generate the grid, mesh, and assign properties.
        *   `void execute_reduced_task(WireGenerator*, WireStructure*, int, MPI_Comm, MPI_Comm)`: A version of `execute_task` that may skip some steps like meshing.
        *   `void write_grid(const char* filename)`: Writes grid point coordinates to a file.
        *   `void write_tetrahedron(const char* filename)`: Writes tetrahedral mesh data to a file.
        *   `void write_gate(const char* filename)`: Writes gate index information to a file.
        *   `void delete_tetrahedron()`: Frees memory associated with tetrahedral mesh data.
    *   **Private Members (Variables)**:
        *   `int NB`: Number of bonds (likely from `WireGenerator`).
        *   `int N3D`: Constant for 3 dimensions.
        *   `int mpi_size, mpi_rank`: MPI communicator size and rank.
        *   `int del_all_var`: Flag for controlling deallocation in destructor.
    *   **Private Members (Functions - a selection)**:
        *   `void make_grid(WireGenerator*, int grid_accuracy)`: Generates 3D grid points.
        *   `void make_1D_grid(WireGenerator*)`: Generates a 1D grid.
        *   `void sort_grid_point(...)`: Sorts and filters grid points.
        *   `void sort_xyz(...)`: Helper for sorting coordinates.
        *   `void write_permittivity(const char*)`: Writes permittivity data.
        *   `void delaunay(...)`: Performs Delaunay triangulation (likely using Qhull).
        *   `double calc_tri_area(...)`, `void calc_vec(...)`, `double calc_tri_volume(...)`, `double calc_max_distance(...)`: Geometric helper functions.
        *   `void find_contact(WireStructure*)`: Identifies contact regions on the grid.
        *   `void find_doping(...)`, `void find_1D_doping(...)`, `void add_doping(...)`: Assigns doping values.
        *   `void find_permittivity(...)`, `void find_1D_permittivity(...)`: Assigns permittivity values.
        *   `void reduce_atom_index()`: Creates a reduced atom index mapping.
        *   `int is_in_gate(...)`, `int is_in_quad3D(...)`, `int is_in_quad2D(...)`: Point-in-region test functions.

## Important Variables/Constants
*   *None defined globally in this file. All variables are members of the `FEMGrid` class.*

## Usage Examples

```c++
// Example for FEMGrid class (conceptual)
// WireGenerator wire_gen;
// WireStructure sim_params;
// // ... initialize wire_gen and sim_params ...

// FEMGrid fem_grid_obj;
// MPI_Comm current_comm = MPI_COMM_WORLD;
// fem_grid_obj.execute_task(&wire_gen, &sim_params, num_procs_poisson,
//                           poisson_solver_type, current_comm, current_comm);
// // fem_grid_obj.grid, fem_grid_obj.tetrahedron etc. are now populated.
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <string.h>`
*   `#include <algorithm>`
*   `#include "WireGenerator.H"`
*   `#include "Utilities.H"`
*   `#include "Blas.H"`
*   `#include "Types.H"`

### External Libraries
*   **MPI**: Used for parallel processing.
*   **Qhull**: Implied by the `delaunay` private method declaration (implementation in `.C` file uses Qhull).
*   **BLAS**: Implied by inclusion of `Blas.H` and usage of BLAS routines in the `.C` file.

### Interactions with Other Project Components
*   This header file declares the `FEMGrid` class, whose methods are implemented in `FEMGrid.C`.
*   The `FEMGrid` class is crucial for the Poisson solver module, as it provides the discretized simulation domain and the spatial distribution of materials and doping.
*   It takes a `WireGenerator` object (from `WireGenerator.H`) and a `WireStructure` object (from `Types.H`) as input to access the device's atomic structure and geometric/material parameters.
*   It uses various helper functions and data structures defined in `Utilities.H` and `Types.H`.

---
*This documentation is auto-generated and may require further manual refinement.*
