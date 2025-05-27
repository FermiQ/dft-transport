# Documentation for `src/WireGenerator.H`

## Overview

This header file defines the `WireGenerator` class. This class is a cornerstone of the OMEN project, responsible for the procedural generation of the atomistic structure of the simulated device, such as a nanowire. Its tasks include defining the crystal unit cell, constructing the overall device geometry by replicating and arranging unit cells, establishing atomic bonds, handling boundary conditions, and incorporating physical features like strain, surface roughness, and alloy disorder.

## Key Components

### Classes/Structs
*   `class WireGenerator`:
    *   **Description**: Manages the creation and representation of the physical device structure at an atomic level.
    *   **Public Members (Functions)**:
        *   `WireGenerator(int plattice_type, int ptransport_type)`: Constructor, likely initializes based on lattice and transport types.
        *   `~WireGenerator()`: Destructor, handles cleanup of allocated resources.
        *   `void execute_simple(WireStructure* nanowire, MPI_Comm comm)`: Main public method to trigger the wire generation process.
        *   `void write_LM(const char* filename)`: Writes the `Layer_Matrix` data to a file.
        *   `void write_TLM(const char* filename)`: Writes the total `Layer_Matrix` (potentially before separation) to a file.
        *   `void write_AM(const char* filename)`: Writes the `Around_Matrix` (surrounding atoms) data to a file.
        *   `void write_positions(const char *filename)`: Writes atom positions and types to a file.
        *   `void get_positions(double* buffer)`: Fills a buffer with atom positions.
        *   `void get_atomtype(double* buffer)`: Fills a buffer with atom types.
        *   `void get_neighbors(double* buffer)`: Fills a buffer with neighbor information.
        *   `void get_uc_positions(double* buffer)`: Fills a buffer with unit cell atom positions.
        *   `void get_uc_atomtype(double* buffer)`: Fills a buffer with unit cell atom types.
        *   `void get_uc_neighbors(double* buffer)`: Fills a buffer with unit cell neighbor information.
        *   `int get_no_bond()`: Returns the number of bonds.
        *   `int get_no_atom()`: Returns the number of atoms.
    *   **Public Members (Variables)**:
        *   `Unit_Cell *unit_cell`: Pointer to the defined unit cell structure.
        *   `double *Layer_Matrix`: Main matrix storing atom coordinates, types, and neighbor indices.
        *   `double *Around_Matrix`: Matrix storing information about surrounding atoms (e.g., oxide).
        *   `double cell_width, cell_area, y_width, Ly, Lz, volume, volume_tot, lenx, leny, lenz, Vol_atom`: Geometric properties of the device/cell.
        *   `double *EMidGap, *EGap, EMGL, EMGR, *atomic_mass, EVmaxL, EVmaxR, ECminL, ECminR`: Electronic and material properties.
        *   `int NLayer, *Lmin, *Lmax, NSlab, *Smin, *Smax, *Neigh_Matrix`: Layer and slab definitions and connectivity.
        *   `int Channel_tot, Around_tot, No_Atom, Around_Atom, Rough_Atom`: Atom counts for different regions.
        *   `int *neighbor_layer, layer_per_slab, l_per_slab_ref, LB_size, RB_size`: Parameters related to layers and boundaries.
        *   `int **index_boundary, *index_channel, **bound_pos, *ch_pos, **bound_conv, *ch_conv, *index_arch, *arch_pos, *arch_conv, *b_shift, *orb_per_at`: Indexing and mapping arrays for atoms.
        *   `int QMfirst, QMlast`: Defines the first and last layer of the quantum mechanical region.
        *   `int freedim, poisson_dim`: Dimensionality parameters.
        *   `int NA, NB, SP, SLM`: Unit cell parameters (Number of Atoms, Number of Bonds, SP orbital count, Stride for LayerMatrix).
        *   `int NBlock, lattice_type, max_neighbors`: Parameters for blocking and lattice.
        *   `WireStructure* nwire`: Pointer to the input `WireStructure` parameters.
        *   `BOUNDARY_ENN** Boundary, **Enn`: Structures holding boundary and layer-to-layer connection information.
    *   **Protected Members (Functions - a selection, many are internal helpers)**:
        *   `void delete_variables()`: Frees memory allocated by the class.
        *   `void check_dimension(WireStructure*)`: Adjusts geometry for 1D/2D simulations.
        *   `void calc_info(WireStructure*)`: Calculates geometric properties.
        *   `void make_unit_cell(...)`: Defines the crystal unit cell.
        *   `void change_orientation(...)`: Orients the unit cell.
        *   `void make_strained_unit_cell(Strain*)`: Applies uniform strain to unit cell.
        *   `void add_strain(...)`: Applies potentially non-uniform strain.
        *   `void make_wire(WireStructure*)`: Generates the main wire structure.
        *   `void generate_roughness(...)`: Adds surface roughness.
        *   `void generate_alloy_disorder(...)`: Introduces alloy disorder.
        *   `void make_connections(WireStructure*)`: Establishes atom connectivity.
        *   `void sort_xyz(...)`: Sorts atoms by coordinates and defines layers.
        *   `void cut_boundary_slab(...)`, `void cut_layer(...)`: Prepares boundary and layer data structures.
    *   **Protected Members (Variables)**:
        *   `int tb, N3D, N2D, N1D`: Tight-binding parameter related, dimension constants.
        *   `int hxmin, hxmax, hymin, hymax, hzmin, hzmax`: Min/max unit cell indices for device generation.
        *   `int mpi_size, mpi_rank`: MPI communicator size and rank.
        *   `int **inv_bound_pos,*inv_ch_pos`: Inverse mapping for boundary/channel atoms.
        *   `int NyFold, NzFold`: Folding factors for periodic dimensions.
        *   `int transport_type`: Type of transport calculation.
        *   `double *XYZH, *HXYZ`: Helper arrays for mapping coordinates to indices.
        *   `double cnt_radius`: Radius for CNT generation.
        *   `MPI_Comm wg_comm`: MPI communicator for wire generation tasks.

## Important Variables/Constants
*   *None defined globally in this file. All relevant variables are members of the `WireGenerator` class.*

## Usage Examples

```c++
// WireGenerator is typically instantiated by the main simulation driver
// WireStructure wire_params; // Assume this is loaded from input
// int lattice = wire_params.lattice_type;
// int transport = wire_params.transport_type;
// WireGenerator generator(lattice, transport);
// generator.execute_simple(&wire_params, MPI_COMM_WORLD);
// int num_atoms = generator.get_no_atom();
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <stdio.h>`
*   `#include <math.h>`
*   `#include <iostream>`
*   `#include <algorithm>`
*   `#include <fstream>`
*   `#include <stdlib.h>`
*   `#include <string.h>`
*   `#include "Utilities.H"`
*   `#include "Blas.H"`
*   Commented out includes: `//#include "Material.H"`, `//#include "AtomStrain.H"`

### External Libraries
*   **MPI**: Used for parallel processing during wire generation.
*   **BLAS**: Likely used for vector/matrix operations via wrappers in `Blas.H`.

### Interactions with Other Project Components
*   This class is central to defining the physical structure simulated in OMEN.
*   It uses and populates structures defined in `Types.H` (e.g., `WireStructure`, `Unit_Cell`, `Atom`, `Bond`, `MAT`, `BOUNDARY_ENN`).
*   It utilizes utility functions from `Utilities.H`.
*   The generated atomic structure (`Layer_Matrix`, `No_Atom`, etc.) is a fundamental input for Hamiltonian construction and subsequent solver stages.

---
*This documentation is auto-generated and may require further manual refinement.*
