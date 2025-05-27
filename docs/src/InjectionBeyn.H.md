# Documentation for `src/InjectionBeyn.H`

## Overview

This header file defines the `InjectionBeyn<T>` class. This class is a concrete implementation of the `Injection<T>` abstract base class. It is designed to calculate injection modes, their k-vectors (eigenvalues of a transfer matrix like formulation), and corresponding eigenvectors for device contacts. The method employed is based on Beyn's contour integral approach for solving nonlinear or polynomial eigenvalue problems. This technique is particularly useful for finding eigenvalues within a specified region in the complex plane, typically a circle or annulus.

The Doxygen comment `/*!  \brief Computes the eigenvalues between two circles around 0 and the corresponding eigenvectors of a polynomial eigenvalue problem */` aptly describes its core mathematical task. The calculated modes are essential for wave-function based quantum transport simulations.

## Key Components

### Classes/Structs
*   `template<class T> class InjectionBeyn : public Injection<T>`:
    *   **Description**: Implements the injection calculation using Beyn's contour integral method for polynomial eigenvalue problems.
    *   **Template Parameter**: `T` (typically `double` or `CPX`).
    *   **Public Members (Functions)**:
        *   `InjectionBeyn(int pslab_per_bc, double pcontour_radius)`: Constructor with basic parameters.
        *   `InjectionBeyn(int pslab_per_bc, double pcontour_radius, double psv, int pNQ, int pNC, int pslc)`: Constructor with more detailed control parameters.
        *   `virtual ~InjectionBeyn()`: Destructor.
        *   `virtual void initialize(int ND, int NP, int NM)`: Implements the interface from `Injection<T>`. Initializes parameters like `NM_input` (number of eigenvalues to seek).
        *   `virtual void calc_kphase(TCSR<T>* D, TCSR<T>* P, TCSR<CPX>* T01, int ND, int NP, CPX* kphase, CPX* Vsurf, double* dEk_dk, int* Ntr, int* ind_Ntr, int* Nref, int* ind_Nref, int sign, int NPROW, int NPCOL, MPI_Comm comm, int* info)`: Implements the interface. Calculates k-phases (eigenvalues) and related properties. This version seems to take block-diagonal (`D`) and off-diagonal (`P`) parts of a larger matrix.
        *   `virtual void calc_kphase_sym(...)`: (Two overloads) Pure virtual functions from base class, not implemented here (would require specific logic for symmetric problems if different from general case).
        *   `virtual void calc_kphase(TCSR<T>* H_all, T* S_all, ...)`: (Two overloads) Pure virtual functions from base class, not implemented here.
        *   `int execute(TCSR<CPX>** A, int NR, int NM, CPX* lambdaR, CPX* YF, int sign, MPI_Comm comm)`: The main computational routine that implements Beyn's method. It constructs matrices `Q1` and `Q2` via contour integration and then solves a generalized eigenvalue problem.
    *   **Private Members (Variables)**:
        *   `double mult_factor, svd_fac, contour_radius, imag_limit`: Control parameters for the algorithm (e.g., SVD tolerance, contour details).
        *   `int mpi_size, mpi_rank, loc_size, loc_rank`: MPI communicator details.
        *   `int NM_input`: Estimated number of eigenvalues inside the contour.
        *   `int slab_per_bc`: Interaction range (degree of polynomial EVP).
        *   `int size_lin_comm`: Number of processors per quadrature point for linear solves.
        *   `int NQDR`: Number of quadrature points per circle.
        *   `int NCRC`: Number of circles for contour integration.
        *   `int parallel_svd`: Flag for using parallel SVD.
    *   **Private Members (Functions)**:
        *   `void sort_eig(...)`: Sorts eigenvalues and corresponding eigenvectors.
        *   `int get_type(...)`: Classifies eigenvalues (propagating, evanescent).
        *   `double calc_derivative(...)`: Calculates derivative dE/dk.
        *   `void init_matrix_single(...)`, `void init_matrix_double(...)`: Initializes matrices for the polynomial eigenvalue problem.
        *   `int parallel_decompose(...)`: Performs parallel SVD or related decomposition.
        *   `int decompose(...)`: Performs sequential SVD or decomposition.
        *   `int svd(...)`: Wrapper for SVD computation.
        *   `int eig(...)`: Wrapper for standard eigenvalue computation.
        *   `void allocate_bcast_H(TCSR<CPX>**& H, int source, MPI_Comm comm)`: Allocates and broadcasts Hamiltonian blocks.
        *   `void get_Q_matrix_int(...)`, `void get_Q_matrix(...)`: Computes matrices Q1, Q2 by numerical integration along contours.
        *   `void create_M_matrix(...)`: Forms the matrix M(z) = sum(A_i * z^i) for a given z.
        *   `void solve_Q_sparse(...)`, `void solve_Q_full(...)`, `void inv_Q_full(...)`: Solves linear systems or inverts matrices involving M(z).
        *   `void init_Y_matrix(...)`: Initializes random matrices Y used in Beyn's method.

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example for InjectionBeyn
// int interaction_range = 2; // slab_per_bc
// double radius_contour = 0.8;
// InjectionBeyn<CPX> injector(interaction_range, radius_contour);
// injector.initialize(matrix_dim, num_points, num_modes_est);

// // Assuming H_blocks (TCSR<CPX>**), NR_val, NM_val etc. are set up
// CPX eigenvalues[NM_val];
// CPX eigenvectors[NR_val * NM_val];
// // injector.execute(H_blocks, NR_val, NM_val, eigenvalues, eigenvectors, sign_val, mpi_comm_val);
// // The actual call would be via the calc_kphase from the base class pointer.
```

## Dependencies and Interactions

### Included Files
*   `<omp.h>`
*   `<mpi.h>`
*   `<iostream>`
*   `<algorithm>`
*   `<fstream>`
*   `<stdlib.h>`
*   `<stdio.h>`
*   `<math.h>`
*   `<time.h>`
*   `<vector>`
*   `#include "Types.H"`
*   `#include "Utilities.H"`
*   `#include "Blas.H"`
*   `#include "ParallelEig.H"`
*   `#include "LinearSolver.H"`
*   `#include "Umfpack.H"` (Conditionally if `HAVE_UMFPACK` is defined)
*   `#include "Injection.H"`

### External Libraries
*   **OpenMP**: For shared-memory parallelism.
*   **MPI**: For distributed-memory parallelism.
*   **BLAS/LAPACK**: Used for dense matrix operations (e.g., `c_zgemm`, `c_zgeev`, `c_zgesdd`).
*   **ScaLAPACK**: Potentially used via `ParallelEig.H` or for parallel SVD.
*   **UMFPACK**: (Conditional) May be used as a sparse linear solver if `HAVE_UMFPACK` is defined.

### Interactions with Other Project Components
*   This class implements the `Injection<T>` interface defined in `Injection.H`.
*   It relies on `TCSR<CPX>` (from `CSR.H`) for sparse matrix representation and `CPX` (from `Types.H`) for complex arithmetic.
*   It uses utility functions from `Utilities.H` and BLAS/LAPACK wrappers from `Blas.H`.
*   It may use parallel eigensolvers from `ParallelEig.H` and linear solvers from `LinearSolver.H` or specific implementations like `Umfpack.H`.
*   The calculated injection terms (eigenvalues `lambdaR` and eigenvectors `YF` from `execute`, or processed into `kphase`, `Vsurf` etc. by `calc_kphase`) are used by the `GetSigma` module to construct self-energies.

---
*This documentation is auto-generated and may require further manual refinement.*
