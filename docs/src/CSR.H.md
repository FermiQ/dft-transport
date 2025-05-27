# Documentation for `src/CSR.H`

## Overview

This header file is central to the OMEN project's handling of sparse matrices. It defines classes for the Compressed Sparse Row (CSR) format:
*   An older, non-templated `CSR` class that stores real and imaginary parts of complex numbers separately.
*   A more modern and flexible templated class `TCSR<T>` for CSR matrices, allowing various data types for matrix elements.

Additionally, this file declares several important enumerations that define types of methods used within the simulation (e.g., SCF methods, solver types, integration techniques) and data structures for passing transport parameters (`transport_parameters`), defining contacts (`contact_type`), and holding results (`result_type`). It also includes a few utility template functions.

## Key Components

### Namespaces
*   `cp2k_methods`: Contains `enum cp2k_method_type`.
*   `lin_solver_methods`: Contains `enum lin_solver_method_type`.
*   `inv_solver_methods`: Contains `enum inv_solver_method_type`.
*   `injection_methods`: Contains `enum injection_method_type`.
*   `real_int_methods`: Contains `enum real_int_method_type`.
*   `transport_methods`: Contains `enum transport_method_type`.

### Enums
*   `cp2k_methods::cp2k_method_type`: Defines methods like `WRITE_OUT`, `LOCAL_SCF`, `TRANSMISSION`, `TRANSPORT`.
*   `lin_solver_methods::lin_solver_method_type`: Defines linear solver types like `SPLITSOLVE`, `SUPERLU`, `MUMPS`, `PARDISO`, etc.
*   `inv_solver_methods::inv_solver_method_type`: Defines matrix inversion solver types like `FULL`, `PEXSI`, `RGF`.
*   `injection_methods::injection_method_type`: Defines injection methods like `EVP`, `BEYN`.
*   `real_int_methods::real_int_method_type`: Defines real-axis integration methods like `GAUSSCHEBYSHEV`, `TRAPEZOIDAL`.
*   `transport_methods::transport_method_type`: Defines transport formalisms like `EQ` (equilibrium), `NEGF`, `GF`, `WF`.

### Structs
*   `struct transport_parameters`: A large structure holding numerous parameters controlling the transport simulation, including method choices, convergence criteria, energy ranges, and solver settings.
*   `struct contact_type`: Defines properties of a contact, such as bandwidth, number of atoms, starting atom index, degrees of freedom, and effective number of electrons.
*   `struct result_type`: Holds results from calculations, like transmission, density of states (DOS), and condition numbers.
*   `struct CSR_Exception`: A simple exception structure for CSR-related errors.

### Classes
*   `class CSR`:
    *   **Description**: An older, non-templated class for CSR sparse matrices. Stores real and imaginary parts of complex numbers in separate `double` arrays.
    *   **Public Members (Functions)**: `CSR(int N, int n_nnz)`, `~CSR()`, `update_diag(double* r_diag, double* i_diag)`, `r_update_diag(double* r_diag)`, `i_update_diag(double* i_diag)`, `get_row_edge()`, `write(const char* filename)`.
    *   **Public Members (Variables)**: `pos`, `size`, `n_nonzeros`, `type`, `double *r_nnz`, `double *i_nnz`, `int *index_i`, `int *index_j`, `int *edge_i`, `int *diag_pos`.
*   `template <class T> class TCSR`:
    *   **Description**: A templated class for CSR sparse matrices, allowing flexibility in data type `T` for matrix elements.
    *   **Public Members (Functions - Selected)**:
        *   Constructors: `TCSR(int N, int n_nnz, int findx)`, `TCSR(TCSR<T>* other_matrix)`, `TCSR(const char* filename)`, `TCSR(cp2k_csr_interop_type& cp2k_matrix, ...)` and several others for various initialization scenarios including MPI distribution and submatrix extraction.
        *   `~TCSR()`: Destructor.
        *   Matrix manipulation: `update_diag(...)`, `get_row_edge()`, `get_diag_pos()`, `mat_vec_mult(...)`, `trans_mat_vec_mult(...)`, `sparse_to_full(...)`, `full_to_sparse(...)`, `sparse_transpose(...)`, `add(...)`, `remove_thr(...)`, `set_to_id()`.
        *   I/O: `write_CSR(const char* filename)`, `write(const char* filename)`, `write_CSR_bin(const char* filename)`.
        *   MPI related: `Bcast(...)`, `scatter(...)`, `reduce(...)`, `reducescatter(...)`, `distribute_back(...)`.
    *   **Public Members (Variables)**: `pos`, `size`, `size_tot`, `n_nonzeros`, `type`, `findx` (Fortran indexing flag), `first_row`, `T *nnz`, `int *index_i` (non-zeros per row), `int *index_j` (column indices), `int *edge_i` (row pointers), `int *diag_pos` (diagonal element positions).

### Template Functions (Free Functions)
*   `template <class T> void full_transpose(int ncol, int nrow, T *B, T* A)`: Transposes a dense matrix.
*   `template <class T> void set_to_zero(int length, T *array)`: Sets all elements of an array to zero.
*   `template <typename T> T fermi(T E, T mu, double temp, int mode)`: Calculates the Fermi-Dirac distribution value or related quantities based on `mode`.

### Macros
*   `LOGCERR`: Macro for logging errors with file and line information: `std::cerr << "ERROR IN LINE " << __LINE__ << " OF FILE " << __FILE__ << std::endl`

## Important Variables/Constants
*   *None defined globally in this file, apart from enum members.*

## Usage Examples

```c++
// Example for TCSR class
// TCSR<CPX> H_matrix(num_orbitals, num_non_zeros, 1); // Fortran-style indexing
// // ... populate H_matrix ...
// H_matrix.write_CSR("hamiltonian.csr");

// Example for fermi function
// double occupation = fermi(energy_level, fermi_energy, temperature, 0);
```

## Dependencies and Interactions

### Included Files
*   `#include <omp.h>`
*   `#include <fstream>`
*   `#include <algorithm>`
*   `#include <complex>`
*   `#include "Blas.H"`
*   `#include "libcp2k.h"` (CP2K interoperability header)

### External Libraries
*   **OpenMP**: For potential parallelization within matrix operations.
*   **CP2K**: Interacts with CP2K data structures via `libcp2k.h` and `cp2k_csr_interop_type`.
*   **BLAS**: Uses BLAS routines for some operations, via `Blas.H`.

### Interactions with Other Project Components
*   Defines the primary sparse matrix formats (`CSR`, `TCSR<T>`) used throughout OMEN for representing Hamiltonians, overlap matrices, Green's functions, etc.
*   The enumerations for method types are used for configuring and controlling simulation workflows.
*   The `transport_parameters` structure is a key data object for passing settings to various computational engines.
*   Relies on `Types.H` (implicitly through `Blas.H` for `CPX` and other headers) for basic data types.
*   Relies on `Blas.H` for underlying BLAS operations used in some `TCSR` methods.

---
*This documentation is auto-generated and may require further manual refinement.*
