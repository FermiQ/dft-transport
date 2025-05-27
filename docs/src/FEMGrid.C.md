# Documentation for `src/FEMGrid.C`

## Overview

This file implements the `FEMGrid` class, which is responsible for generating and managing the Finite Element Method (FEM) grid used in OMEN, primarily for electrostatic Poisson solves. Its key tasks include:
1.  Creating a grid of points based on the atomic structure provided by `WireGenerator`. This grid may include points at atomic sites and potentially additional points for finer discretization, especially around bonds.
2.  Performing Delaunay triangulation on the generated grid points using the Qhull library to create a mesh of tetrahedra. This mesh is essential for FEM calculations.
3.  Assigning physical properties to the grid points or tetrahedra, such as doping concentrations and material permittivity.
4.  Identifying grid points associated with device features like gates, source, drain, and ground contacts.

## Key Components

### Functions
*(These are member functions of the `FEMGrid` class, declared in `FEMGrid.H`)*
*   `FEMGrid()`: Constructor. Initializes MPI rank/size and default values.
*   `~FEMGrid()`: Destructor. Frees allocated memory for grid, doping, permittivity, and various index arrays.
*   `void delete_tetrahedron()`: Specifically frees memory related to the Delaunay tetrahedra.
*   `void execute_task(WireGenerator *Wire, WireStructure *nanowire, int max_proc_poisson, int poisson_solver, MPI_Comm po_comm, MPI_Comm vg_comm)`: The main public method to orchestrate grid generation, Delaunay triangulation, and property assignment.
*   `void execute_reduced_task(WireGenerator *Wire, WireStructure *nanowire, int max_proc_poisson, MPI_Comm po_comm, MPI_Comm vg_comm)`: Similar to `execute_task`, but may skip computationally intensive steps like Delaunay triangulation if only grid properties are needed.
*   `void make_1D_grid(WireGenerator *Wire)`: Generates a simplified 1D grid based on atomic layer positions.
*   `void make_grid(WireGenerator *Wire, int grid_accuracy)`: Generates 3D grid points. It starts with atom positions and adds points along bonds for finer resolution based on `grid_accuracy`.
*   `void reduce_atom_index()`: Creates a mapping from the full atom indices to a reduced set, likely for atoms included in the Poisson domain.
*   `void sort_grid_point(WireGenerator *Wire, double *gpoints, int NP, int No_Atom, int Around_Atom, double xmax)`: Sorts the generated grid points (using `sort_xyz`) and filters them, populating the main `grid` array and `atom_index` mappings.
*   `void find_contact(WireStructure *nanowire)`: Identifies and stores indices of grid points that fall within gate, ground, source, or drain regions defined in `WireStructure`.
*   `void find_permittivity(WireStructure *nanowire)`: Assigns permittivity values to each grid point based on the material regions defined in `WireStructure`.
*   `void find_1D_permittivity(WireStructure *nanowire)`: Simplified permittivity assignment for 1D grids.
*   `void find_doping(WireStructure *nanowire, double Vol_atom, int No_Atom)`: Assigns doping concentrations to grid points based on defined doping regions or by reading from a file.
*   `void find_1D_doping(WireStructure *nanowire)`: Simplified doping assignment for 1D grids.
*   `void add_doping(WireStructure *nanowire, double Vol_atom, int No_Atom)`: Adds doping contributions from analytically defined doping domains (e.g., Gaussian profiles).
*   `int is_in_gate(GATE *gate, double *p0, double max_dist)`: Checks if a given point `p0` is located on the surface of a `GATE` object.
*   `void sort_xyz(int NP, XYZPOS* xyz, int *index, int* NL, int* LMI, int* LMA)`: Sorts an array of `XYZPOS` structures first by x, then y, then z coordinates. It also identifies layer boundaries (`LMI`, `LMA`).
*   `void delaunay(int update_tetra, char* tetra_file, int update_fitness, int max_proc_poisson, int poisson_solver, MPI_Comm po_comm, MPI_Comm vg_comm)`: Performs 3D Delaunay triangulation of the grid points using the Qhull library. It can also read tetrahedral data from a file.
*   `void write_grid(const char *filename)`: Writes the grid coordinates to a file.
*   `void write_tetrahedron(const char *filename)`: Writes the tetrahedral mesh data to a file.
*   `void write_gate(const char *filename)`: Writes gate index information to a file.
*   `void write_permittivity(const char *filename)`: Writes permittivity values at grid points to a file.
*   `double calc_tri_area(double* p1, double* p2, double* p3, int ND)`: Calculates the area of a triangle.
*   `void calc_vec(double* v, double* p1, double* p2, double* p3)`: Calculates the normal vector to a plane defined by three points.
*   `double calc_tri_volume(double *p1, double *p2, double *p3, double *p4)`: Calculates the volume of a tetrahedron.
*   `double calc_max_distance(double *p1, double *p2, double *p3, double *p4)`: Calculates the maximum edge length in a tetrahedron.
*   `double calc_tri_volume(double A, double* p0, double* p1, double* v)`: Calculates volume using face area and normal vector component.
*   `void calc_doping_info(WireStructure *nanowire)`: Pre-calculates geometric properties of doping regions.
*   `double calc_quad_area(double* p1, double* p2, double* p3, double* p4, int ND)`: Calculates area of a quadrilateral.
*   `double calc_quad_volume(DOPING* doping)`: Calculates volume of a general hexahedron-like doping region.
*   `int is_in_quad3D(DOPING* doping, double* p)`: Checks if a point is inside a 3D doping region.
*   `int is_in_quad3D(MAT* mat, double* p)`: Checks if a point is inside a 3D material region.
*   `int is_in_quad2D(GATE* gate, double* p)`: Checks if a point is inside a 2D gate region (on a plane).
*   `double check_volume(MAT* mat, double* p)`: Calculates volume formed by a point and material region faces (used for point-in-polyhedron test).
*   `void check_dimension(WireGenerator *Wire, WireStructure *nanowire)`: Adjusts 2D geometries for 3D simulation context if needed.

### Classes/Structs
*   *None defined in this file. It implements methods for the `FEMGrid` class declared in `FEMGrid.H`.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// FEMGrid is typically used by a higher-level simulation setup routine.
// WireGenerator wire_gen_instance(...);
// WireStructure device_params;
// // ... populate wire_gen_instance and device_params ...

// FEMGrid fem_grid_generator;
// MPI_Comm poisson_comm, voltage_gate_comm;
// // ... initialize communicators ...
// fem_grid_generator.execute_task(&wire_gen_instance, &device_params,
//                                 max_procs_for_poisson, poisson_solver_type,
//                                 poisson_comm, voltage_gate_comm);
// // fem_grid_generator now contains the grid, tetrahedra, doping, etc.
```

## Dependencies and Interactions

### Included Files
*   `#include "FEMGrid.H"`
*   `#include <libqhull.h>` (Qhull library for Delaunay triangulation)
*   `#include <qset.h>` (Qhull library)

### External Libraries
*   **Qhull**: Used for Delaunay triangulation (`qh_new_qhull`, `qh_freeqhull`, etc.).
*   **MPI**: Used for parallel processing and data distribution.
*   **BLAS**: Implied by the use of `c_dcopy`, `c_daxpy`, `c_ddot`, `c_dnrm2`, `c_dscal` (likely from `Blas.H` included via `FEMGrid.H` or `WireGenerator.H`).

### Interactions with Other Project Components
*   This file implements the `FEMGrid` class, declared in `FEMGrid.H`.
*   It takes a `WireGenerator` object (from `WireGenerator.H`) and a `WireStructure` object (from `Types.H`) as input to access the atomic structure and device parameters.
*   The generated FEM grid, tetrahedral mesh, doping profiles, and permittivity maps are crucial inputs for subsequent Poisson equation solvers.
*   Uses utility functions (e.g., `sort_xyz`, `min_vec`, `max_vec`) which might be from `Utilities.H`.
*   Uses data structures like `GATE`, `MAT`, `DOPING`, `XYZPOS` from `Types.H`.

---
*This documentation is auto-generated and may require further manual refinement.*
