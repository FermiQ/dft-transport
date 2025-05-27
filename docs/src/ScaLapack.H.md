# Documentation for `src/ScaLapack.H`

## Overview

This header file provides C++ wrapper functions for various ScaLAPACK (Scalable Linear Algebra PACKage) and BLACS (Basic Linear Algebra Communication Subprograms) routines. It enables the use of these high-performance Fortran libraries, which are typically Fortran-based, from within C++ code. The wrappers handle aspects like name mangling (`fortran_name`) and provide C++-friendly interfaces. This is crucial for performing distributed memory parallel linear algebra operations within the OMEN project.

## Key Components

### Functions

This file primarily consists of:
1.  `extern "C"` declarations of Fortran ScaLAPACK and BLACS routines.
2.  Inline C++ wrapper functions (prefixed with `c_` or `C`) that call these Fortran routines.

#### Fortran Routine Declarations (subset, for brevity)
*   `fortran_name(dgehrd,DGEHRD)(...)`
*   `fortran_name(dlacpy,DLACPY)(...)`
*   `fortran_name(pdlamch,PDLAMCH)(...)`
*   `fortran_name(numroc,NUMROC)(...)`
*   `fortran_name(descinit,DESCINIT)(...)`
*   `fortran_name(pdsyev,PDSYEV)(...)`
*   `fortran_name(pzgemm,PZGEMM)(...)`
*   `fortran_name(blacs_pinfo,BLACS_pinfo)(...)`
*   `fortran_name(blacs_gridinit,BLACS_GRIDINIT)(...)`
*   *(... and many others for various linear algebra operations like LU decomposition, matrix multiplication, eigenvalue problems, etc.)*

#### C++ Wrapper Functions (subset, for brevity)
*   `inline void c_dgehrd(int N,int ILO,int IHI,double *A,int LDA,double *TAU,double *WORK, int LWORK,int *INFO)`
*   `inline void c_dlacpy(char UPLO,int M,int N,double *A,int LDA,double *B,int LDB)` (if `__BLAS` is not defined)
*   `inline void c_zlacpy(char UPLO,int M,int N,CPX *A,int LDA,CPX *B,int LDB)` (if `__BLAS` is not defined)
*   `inline double c_pdlamch(int ictxt,char cmach)`
*   `inline int c_numroc(int N,int NB,int IPROC,int ISRCPROC,int NPROCS)`
*   `inline void c_descinit(int *DESC,int M,int N,int MB,int NB,int IRSRC,int ICSRC,int ICTXT, int LLD, int *INFO)`
*   `inline void c_pdsyev(char JOBZ,char UPLO,int N,double *A,int IA,int JA,int *DESCA,double *W, double *Z,int IZ,int JZ,int *DESCZ,double *WORK,int LWORK,int *INFO)`
*   `inline void c_pzgemm(char transa,char transb,int m,int n,int k,CPX alpha,CPX *a,int ia, int ja,int *desc_a,CPX *b,int ib,int jb,int *desc_b,CPX beta, CPX *c,int ic,int jc,int *desc_c)`
*   `inline void Cblacs_pinfo(int *MYPNUM,int *NPROCS)`
*   `inline void Cblacs_gridinit(int *ICONTXT,char *ORDER,int NPROW,int NPCOL)`
*   *(... and many other C++ wrappers corresponding to the Fortran routines)*

#### Custom/Utility Functions
*   `void my_dgeev(...)`
*   `void pdgeev(...)`
*   `void pzgeev(...)`
*   `void pdgeev_driver(...)`
*   `void pzgeev_driver(...)`
*   `void pdgemm_driver(...)`
*   `void pzgemm_driver(...)`
*   `void distribute(...)`
*   `void zdistribute(...)`
*   `void receive(...)`
*   `void zreceive(...)`
*   `void insert_block(...)`
*   `void zinsert_block(...)`
*   `void get_hess(...)`
*   `void get_zhess(...)`
*   `void get_blocking_factors(...)`

### Classes/Structs
*No classes or structs are defined in this file.*

### Macros
*   `__SCALAPACK`: Header guard.
*   `__BLAS`: Conditional compilation macro, suggesting some BLAS wrappers might be excluded if `__BLAS` is defined elsewhere.
*   `fortran_name(lower,UPPER)`: Macro likely used to handle Fortran name mangling differences across compilers/systems.

## Important Variables/Constants
*No global variables or constants are explicitly defined in this file. Constants are typically passed as arguments to functions.*

## Usage Examples

```c++
// Conceptual example of using a ScaLAPACK wrapper
// Assuming context, descriptors, matrices A, B, C are set up
// For example, performing a parallel matrix multiplication:
// CPX alpha = {1.0, 0.0};
// CPX beta = {0.0, 0.0};
// c_pzgemm('N', 'N', m, n, k, alpha, mat_A, ia, ja, desc_A,
//          mat_B, ib, jb, desc_B, beta, mat_C, ic, jc, desc_C);
```

## Dependencies and Interactions

### Included Files
*   `#include <mpi.h>`
*   `#include <iostream>`
*   `#include <stdio.h>`
*   `#include <fstream>`
*   `#include <time.h>`
*   `#include "Blas.H"`
*   `#include "Utilities.H"`

### External Libraries
*   **ScaLAPACK**: The primary purpose of this file is to wrap ScaLAPACK routines.
*   **BLACS**: Wrappers for BLACS routines are also included, which handle communication for ScaLAPACK.
*   **MPI**: Indicated by `#include <mpi.h>` and the parallel nature of ScaLAPACK/BLACS.
*   **BLAS**: Indicated by `#include "Blas.H"` and the conditional compilation for `dlacpy`/`zlacpy`.

### Interactions with Other Project Components
*   Provides fundamental linear algebra capabilities for other parts of the OMEN project that deal with large-scale matrix operations in a distributed memory environment.
*   Depends on `Blas.H` for some BLAS operations and `Utilities.H` for utility functions (like `CPX` type).
*   The custom driver and utility functions (e.g., `pdgeev_driver`, `distribute`) suggest higher-level operations built upon the ScaLAPACK wrappers for use within the project.

---
*This documentation is auto-generated and may require further manual refinement.*
