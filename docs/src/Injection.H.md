# Documentation for `src/Injection.H`

## Overview

This header file defines the `Injection<T>` class. This class serves as an abstract base class (interface) for various methods that calculate injection terms (often self-energies or related quantities representing incident waves from contacts) used in quantum transport simulations, particularly those based on wave function approaches. It establishes a common interface that specific injection calculation algorithms (like Beyn or IEV) must implement.

## Key Components

### Classes/Structs
*   `template<class T> class Injection`:
    *   **Description**: An abstract base class for injection term calculations. It cannot be instantiated directly; derived classes must implement its pure virtual functions.
    *   **Template Parameter**: `T` (typically `double` or `CPX` for the matrix element types).
    *   **Public Members (Functions)**:
        *   `Injection()`: Default constructor.
        *   `Injection(int, double)`: Another constructor (parameters not named, usage unclear from header).
        *   `virtual ~Injection()`: Virtual destructor.
        *   `virtual void initialize(int NBlock, int Bsize, int tb) = 0;`: Pure virtual function to initialize the injection calculation, likely with block and matrix parameters.
        *   `virtual void calc_kphase(TCSR<T>* H0, TCSR<T>* H1, TCSR<CPX>* Sigma, int NR, int n_eig, CPX* lambda, CPX* X0, double* vel, int* npro, int* ndec, int* degen, int* master_nodes, int inj_sign, int NE, MPI_Comm comm, int* info) = 0;`: Pure virtual function to calculate k-dependent phase factors (eigenvalues `lambda`), eigenvectors (`X0`), velocities (`vel`), and counts of propagating/decaying modes. This version likely takes sparse `H0` and `H1`.
        *   `virtual void calc_kphase_sym(TCSR<T>* H0, TCSR<T>* H1, TCSR<CPX>* Sigma, int NR, int n_eig, CPX* lambda_p, CPX* lambda_m, CPX* X0_p, CPX* X0_m, double* vel_p, double* vel_m, int* npro_p, int* ndec_p, int* degen_p, int* npro_m, int* ndec_m, int* degen_m, int* master_nodes, int inj_sign, int NE, MPI_Comm comm, int* info) = 0;`: Pure virtual function, similar to `calc_kphase` but possibly for symmetric systems or to compute both positive and negative k-solutions.
        *   `virtual void calc_kphase(TCSR<T>* H_all, T* S_all, TCSR<CPX>* Sigma, int NR, int n_eig, CPX* lambda, CPX* X0, double* vel, int* npro, int* ndec, int* degen, int* master_nodes, int inj_sign, int NE, MPI_Comm comm, int* info) = 0;`: Overload of `calc_kphase`, possibly taking a full Hamiltonian `H_all` and overlap `S_all` in different formats.
        *   `virtual void calc_kphase_sym(TCSR<T>* H_all, T* S_all, TCSR<CPX>* Sigma, int NR, int n_eig, CPX* lambda_p, CPX* lambda_m, CPX* X0_p, CPX* X0_m, double* vel_p, double* vel_m, int* npro_p, int* ndec_p, int* degen_p, int* npro_m, int* ndec_m, int* degen_m, int* master_nodes, int inj_sign, int NE, MPI_Comm comm, int* info) = 0;`: Overload of `calc_kphase_sym`.

### Functions
*   *None defined outside the class.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// This is an abstract class and cannot be instantiated directly.
// Derived classes like InjectionBeyn or InjectionIEV would be used:

// InjectionBeyn<CPX> injector_beyn; // Or InjectionIEV
// injector_beyn.initialize(...);
// injector_beyn.calc_kphase(...);
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <iostream>`
*   `#include <algorithm>`
*   `#include <fstream>`
*   `#include <stdlib.h>`
*   `#include <stdio.h>`
*   `#include <math.h>`
*   `#include <time.h>`
*   `#include "Types.H"`
*   `#include "Utilities.H"`
*   `#include "Blas.H"`
*   `#include "ScaLapack.H"`

### External Libraries
*   **MPI**: Used for parallel computations, as indicated by `MPI_Comm`.
*   **BLAS**: Implied by `Blas.H`, likely used by concrete implementations for matrix operations.
*   **ScaLAPACK**: Implied by `ScaLapack.H`, potentially used by concrete implementations for distributed linear algebra.

### Interactions with Other Project Components
*   This header defines an abstract interface for injection calculations.
*   Concrete derived classes (e.g., `InjectionBeyn`, `InjectionIEV`) will implement these pure virtual functions.
*   It uses `TCSR<T>` (from `CSR.H`) and `CPX` (from `Types.H`) for matrix and data representation.
*   The injection terms calculated by implementations of this interface are crucial for the `GetSigma` module and subsequently for wave-function based transport solvers.

---
*This documentation is auto-generated and may require further manual refinement.*
