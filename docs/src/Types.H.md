# Documentation for `src/Types.H`

## Overview

This header file is a central repository for custom data types, structures (structs), and global constants used throughout the OMEN project. It defines the fundamental building blocks for representing physical entities like atoms, bonds, and unit cells, as well as structures for simulation parameters, geometric descriptions, material properties, contact information, and various numerical settings. Many other files in the project will include `Types.H` to use these common definitions.

## Key Components

### Typedefs
*   `typedef complex<double> CPLXType;`: Defines `CPLXType` as an alias for `std::complex<double>`.
*   `typedef CPLXType CPX;`: Defines `CPX` as a shorter alias for `CPLXType` (complex double-precision numbers).
*   `typedef CPX *CPXp;`: Defines `CPXp` as a pointer to a `CPX` type.
*   `typedef CPX (*CPXpfn)(CPX);`: Defines `CPXpfn` as a pointer to a function that takes a `CPX` and returns a `CPX`.

### Structs
*   `struct POINT3D`: Represents a point in 3D space with `double coord[3]`.
*   `struct POINT2D`: Represents a point in 2D space with `double coord[2]`.
*   `struct Axis`: Represents an axis vector with `double vec[3]`.
*   `struct Bond`: Describes a chemical bond, including its vector, SP3 hybridization parameters, equilibrium length `d0`, and neighbor index.
*   `struct Atom`: Represents an atom with its type, 3D coordinates, and internal displacement.
*   `struct at_type`: Contains a pointer to an array of `Bond` pointers.
*   `struct Unit_Cell`: Defines the unit cell, including atom types, atom positions, lattice vectors, Brillouin zone points, strain information, and passivation details.
*   `struct Strain`: Defines strain parameters, including `on` flag and components of the strain tensor (`Eps_xx`, etc.), and `zeta`.
*   `struct StrainDomain`: Defines a domain with specific strain characteristics.
*   `struct Contact`: Describes contact properties, including activity, type, barrier height, and virtual conduction band energy.
*   `struct Roughness`: Parameters for surface roughness, including `on` flag, seed, type, RMS, and correlation length.
*   `struct Alloy`: Parameters for alloy materials, including `on` flag, seed, and composition.
*   `struct MAT`: Describes a material region, including geometry (points, radius, face areas, volume), ID, type, and cross-section.
*   `struct SURFACE`: Describes a surface, including 2D points, radius, face area, ID, type, and cross-section.
*   `struct DOPING`: Describes a doping region, similar to `MAT` but includes doping concentrations `NA` (acceptor) and `ND` (donor).
*   `struct DopingDomain`: Defines a domain with specific doping characteristics, including concentration and slope.
*   `struct GATE`: Describes a gate terminal, including geometry (points, radius, angle), face area, and type.
*   `struct WireStructure`: A large structure holding comprehensive information about the simulated wire, including file paths, dimensions, folding parameters, material properties, strain, roughness, alloy info, contacts, QM region details, update flags, grid parameters, gate information, doping profiles, and VFF relaxation parameters. This is a central data structure for device definition.
*   `struct XYZPOS`: Represents a 3D position with an associated index.
*   `struct IJPOS`: Represents a 2D grid-like position (i, j) with an associated index.
*   `struct BOUNDARY_ENN`: Related to boundary conditions, possibly for nearest-neighbor interactions, including potential arrays, cell dimensions, and neighbor counts.
*   `struct CONNEC`: Represents a connection with a neighbor index and an overall index.
*   `struct ENERGY`: Defines parameters for energy discretization in calculations, including limits, offsets, energy steps, and maximum number of points.
*   `struct PARAM`: Holds various simulation parameters, including material name, table file, strain model, k-space sampling (Nk, Nky, Nkz), solver types, temperature, Poisson solver settings, directory paths, and transport settings.
*   `struct FermiStructure`: Holds data for Fermi calculations, including energy levels, k-space discretization, number of modes, temperature, spin factor, cell area, and derivative flag.
*   `struct VOLTAGE`: Defines voltage sweep parameters for gates (Vgmin, Vgmax), source (Vsmin, Vsmax), drain (Vdmin, Vdmax), and temperature (Tmin, Tmax).

### Macros
*   `PI`: Defines the mathematical constant Pi.
*   `ROWBLOCK`, `COLBLOCK`: Likely default block sizes for matrix operations.
*   `MAX_COMM`: Maximum number of commands or communicators.
*   `INF`: A large number representing infinity.
*   `SEP_TASK`: A task separation identifier or limit.
*   `tollim`: A tolerance limit for numerical comparisons.
*   `scale_norm`: A scaling factor for normalization.
*   `max_diff_gate`: Maximum number of different gates.
*   `fortran_name(x,y)`: A macro to handle Fortran name mangling, with different definitions based on preprocessor flags (`Add_`, `NoChange`, `UpCase`).

## Important Variables/Constants
*The macros listed above serve as global constants (e.g., `PI`, `INF`, `tollim`). No global variables are declared in this file.*

## Usage Examples

```c++
// Example for CPX type
CPX complex_number(1.0, -1.0); // Represents 1.0 - 1.0i

// Example for a struct
Atom my_atom;
my_atom.type = 1;
my_atom.coord[0] = 0.0;
my_atom.coord[1] = 0.1;
my_atom.coord[2] = 0.2;

WireStructure device_params;
// ... populate device_params ...
```

## Dependencies and Interactions

### Included Files
*   `#include <math.h>`
*   `#include <complex>`

### External Libraries
*   *None directly included, relies on standard C++ libraries.*

### Interactions with Other Project Components
*   This file is fundamental to the OMEN project. Its defined types and structures are expected to be used by nearly all other modules for data representation and parameter configuration.
*   Files dealing with geometry, material properties, simulation setup, numerical calculations, and specific algorithms (like solvers) will heavily depend on the definitions in `Types.H`.

---
*This documentation is auto-generated and may require further manual refinement.*
