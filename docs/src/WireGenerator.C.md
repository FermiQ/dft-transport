# Documentation for `src/WireGenerator.C`

## Overview

This file provides the implementation for the `WireGenerator` class. The primary responsibility of this class is to construct the atomistic representation of the simulated device, typically a nanowire or a similar nanostructure. This involves several steps:
1.  Defining the fundamental crystal unit cell based on the specified lattice type (e.g., zincblende, wurtzite, graphene).
2.  Orienting and replicating this unit cell to form the basic device geometry.
3.  Establishing connections (bonds) between atoms.
4.  Handling geometric features, material interfaces, and boundaries.
5.  Applying physical effects such as strain, surface roughness, and alloy disorder.
6.  Managing data structures that describe the atoms, their positions, types, and connectivity (`Layer_Matrix`).
7.  Interfacing with MPI for distributed generation of large structures.

The note at the beginning indicates that matrices used in BLAS and LAPACK are stored in Fortran style (column-major).

## Key Components

### Functions
*(These are member functions of the `WireGenerator` class)*
*   `WireGenerator(int plattice_type, int ptransport_type)`: Constructor. Initializes default parameters based on lattice type.
*   `~WireGenerator()`: Destructor. Frees dynamically allocated memory, including the unit cell structure and various matrices.
*   `void delete_variables()`: Helper function to deallocate memory for various member arrays (e.g., `Neigh_Matrix`, `Boundary` structures).
*   `void execute_simple(WireStructure* nanowire, MPI_Comm int_comm)`: The main public method that orchestrates the wire generation process.
*   `void make_connections(WireStructure* nanowire)`: Calculates and stores neighbor information for each atom in the `Layer_Matrix`.
*   `void make_wire(WireStructure* nanowire)`: Generates the atomic positions for the wire based on the unit cell and defined geometric regions.
*   `void bcast_wire_info(int Nx, int Ny, int Nz)`: Broadcasts essential wire information (atom count, `Layer_Matrix`, `XYZH` coordinates, layer limits) to all MPI ranks.
*   `void bcast_oxide_info()`: Broadcasts information about surrounding/oxide atoms.
*   `void exchange_wire_info(int At_local, double *LM_local, double *XYZH_local, int max_rank)`: Gathers local atom data from different MPI ranks to the root rank and sorts it.
*   `void exchange_oxide_info(int Around_local, int Rough_local, double *AM_local, int max_rank)`: Gathers surrounding/oxide atom data.
*   `void check_periodicity(WireStructure *nanowire)`: Modifies the `Layer_Matrix` to implement periodic boundary conditions by adding atoms and connections at the boundaries.
*   `void update_periodicity(WireStructure *nanowire)`: Updates internal structures related to periodicity after changes to the atom list.
*   `void hydrogen_passivation(WireStructure *nanowire)`: Adds hydrogen atoms to passivate dangling bonds on the surface of the wire.
*   `void roll_cnt()`: Special function to generate coordinates for a carbon nanotube by "rolling" a graphene sheet.
*   `void make_bulk_layer_matrix(WireStructure *nanowire, int *no_orb)`: Creates a `Layer_Matrix` for bulk material simulations.
*   `void replicate_unit_cell(int replicate_unit_cell, WireStructure *nanowire)`: Replicates a chosen unit cell segment throughout the device length.
*   `void read_atom_position(int read_atom_pos, WireStructure *nanowire)`: Reads atom positions from an external file and reconstructs connectivity.
*   `void get_orbital_per_atom(int *no_orb)`: Calculates the cumulative orbital count for each atom.
*   `void generate_roughness(WireStructure *nanowire, int sample_id)`: Introduces surface roughness by modifying atom types near material interfaces based on a random process.
*   `void get_face_info(...)`, `int is_in_face(...)`, `void generate_random_surface(...)`, `void get_face_center(...)`, `void get_acv_value(...)`, `void eig(...)`: Helper functions for surface roughness generation.
*   `void generate_alloy_disorder(WireStructure *nanowire)`: Randomly changes atom types to simulate alloy disorder based on a specified composition.
*   `void init_variables(WireStructure* nanowire)`: Initializes various arrays and parameters based on the generated `Layer_Matrix`, including layer definitions (`Lmin`, `Lmax`), slab definitions (`Smin`, `Smax`), and boundary structures.
*   `void get_cell_width()`: Calculates the width of the unit cells at the boundaries.
*   `int make_slab(WireStructure* nanowire)`: Determines the number of atoms in a primary slab of the device.
*   `void check_dimension(WireStructure* nanowire)`: Adjusts device dimensions for 1D or 2D simulations by extending boundaries.
*   `double get_length(int axis_ind)`: Calculates the length of the unit cell along a given crystal axis.
*   `void calc_info(WireStructure* nanowire)`: Calculates geometric information like face areas and volumes for defined material regions.
*   `void find_3D_boundary(...)`, `void find_2D_boundary(...)`: Determines the extent of the simulation domain in unit cell coordinates.
*   `void check_uc_axis(...)`: Identifies the primary unit cell axes for 2D calculations.
*   `void make_unit_cell(const char* first_atom, double a0, double c0, double u0)`: Defines the atomic positions, lattice vectors, bond vectors, and neighbor information for various crystal structures (e.g., zincblende, wurtzite, graphene).
*   `void change_orientation(double* x, double* y, double* z, double dsp3)`: Rotates the unit cell according to specified orientation vectors and updates bond parameters.
*   `void get_d_matrix(double *d_matrix,double l,double m,double n)`: Calculates a rotation matrix for d-orbitals (likely for tight-binding parameter transformation, though not directly used within `WireGenerator`'s main flow).
*   `void make_strained_unit_cell(Strain *strain)`: Applies a uniform strain tensor to the unit cell coordinates and bond vectors.
*   `double calc_tri_area(...)`, `double calc_quad_area(...)`, `void calc_vec(...)`, `double calc_tri_volume(...)`, `double calc_quad_volume(...)`: Helper functions for geometric calculations.
*   `int is_in_quad3D(MAT* mat, double* p)`, `int is_in_quad2D(SURFACE* surf, double* p)`: Functions to check if a point `p` is inside a defined 3D material region or 2D surface.
*   `void sort_xyz(XYZPOS* xyz, int *index, int* NL, int* LMI, int* LMA)`: Sorts atoms primarily by x, then y, then z coordinates, and determines layer boundaries (`LMI`, `LMA`).
*   `void cut_boundary_slab(WireStructure* nanowire)`: Extracts and prepares information for the boundary slabs (`Boundary` structures).
*   `void update_boundary_slab(WireStructure* nanowire)`: Updates boundary slab information after modifications like alloy disorder.
*   `void cut_layer(WireStructure* nanowire)`: Populates `Enn` structures for each layer, detailing intra-layer and inter-layer connections.
*   `void separate_dimension(WireStructure *nanowire)`: Classifies atoms based on their position relative to periodic boundaries (for 1D/2D simulations) using `index_channel`.
*   `void boundary_dimension(int *NBOUND,int *NCH,int NDim,int IB)`: Helper for `separate_dimension` specifically for boundary atoms.
*   `void convert_position(WireStructure *nanowire)`: Adjusts atom indices and positions after dimensional separation.
*   `void add_strain(Strain *strain, int no_strain_domain, StrainDomain **strain_domain, int update_at)`: Applies strain to the entire wire structure, potentially using multiple strain domains.
*   `void strain_coordinate(...)`, `void get_strain_matrix(...)`: Helper functions for applying strain.
*   `void get_boundary_size(WireStructure *nanowire)`: Calculates the size of boundary regions needed for solvers like SPIKE.
*   `void get_qm_region(WireStructure* nanowire)`: Defines the start and end layers of the quantum mechanical simulation region.
*   `void write_LM(const char *filename)`, `void write_TLM(const char *filename)`, `void write_AM(const char *filename)`, `void write_positions(const char *filename)`: Functions to write atomistic data to files for debugging or visualization.

### Classes/Structs
*   *None defined in this file. This file implements methods of the `WireGenerator` class declared in `WireGenerator.H`.*

## Important Variables/Constants
*   *None defined globally in this file. Member variables of `WireGenerator` are used extensively.*

## Usage Examples

```c++
// WireGenerator is typically instantiated and used by the main simulation setup.
// WireStructure params; // Assume populated from input file
// WireGenerator generator(params.lattice_type, params.transport_type);
// MPI_Comm some_communicator = MPI_COMM_WORLD;
// generator.execute_simple(&params, some_communicator);
// // The Layer_Matrix and other members of 'generator' now contain the device structure.
```

## Dependencies and Interactions

### Included Files
*   `#include "WireGenerator.H"`

### External Libraries
*   **BLAS**: Used for various vector and matrix operations (e.g., `c_dcopy`, `c_daxpy`, `c_dgemv`, `c_dnrm2`, `c_dscal`, `c_ddot`, `c_dsyev`, `c_dgetrf`, `c_dgetrs`). These are likely C-wrapped Fortran BLAS/LAPACK routines.
*   **MPI**: Used for parallel generation of large structures and data distribution (`MPI_Comm_size`, `MPI_Comm_rank`, `MPI_Send`, `MPI_Recv`, `MPI_Bcast`, `MPI_Barrier`).

### Interactions with Other Project Components
*   This file provides the implementation for the `WireGenerator` class declared in `WireGenerator.H`.
*   It heavily relies on data structures defined in `Types.H` (e.g., `WireStructure`, `Unit_Cell`, `Atom`, `Bond`, `MAT`, `Strain`, `XYZPOS`, `BOUNDARY_ENN`).
*   It uses BLAS wrapper functions, likely declared in `Blas.H`.
*   The generated `Layer_Matrix` and other geometric data are fundamental inputs for subsequent Hamiltonian construction and transport calculation modules.

---
*This documentation is auto-generated and may require further manual refinement.*
