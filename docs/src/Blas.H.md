# Documentation for `src/Blas.H`

## Overview

This header file serves as a comprehensive interface to standard Basic Linear Algebra Subprograms (BLAS) and Linear Algebra Package (LAPACK) routines. It provides C++ wrapper functions that allow seamless calling of these routines, which are typically implemented in Fortran. The wrappers handle aspects like name mangling (via the `fortran_name` macro, not explicitly shown but implied by typical BLAS/LAPACK usage) and provide C++-friendly interfaces.

The file includes:
1.  `extern "C"` declarations for a multitude of BLAS (Level 1, 2, 3) and LAPACK (solver, eigenvalue, SVD routines) functions for both real (`double`) and complex (`CPX`) data types.
2.  Inline C++ wrapper functions (typically prefixed with `c_`) that call these Fortran routines.
3.  C++ template functions (e.g., `c_tcopy`, `c_tgemm`) for type-generic operations, which internally call the appropriate `double` or `CPX` specific wrappers.
4.  Conditional inclusions and template specializations for GPU-accelerated operations using MAGMA and custom CUDA utilities (if `HAVE_SPLITSOLVE` is defined).

This file is crucial for performing high-performance numerical linear algebra within the OMEN project.

## Key Components

### Namespaces
*   `MYBLAS`: Encapsulates all declarations and definitions in this file.

### Functions

#### Fortran Routine Declarations (`extern "C"`)
*(A representative subset is listed due to the large number of routines)*
*   **BLAS Level 1**: `fortran_name(dcopy,DCOPY)`, `fortran_name(zaxpy,ZAXPY)`, `fortran_name(dnrm2,DNRM2)`, `fortran_name(ddot,DDOT)`, `fortran_name(zdotc,ZDOTC)`, `fortran_name(dasum,DASUM)`, `fortran_name(dzasum,DZASUM)`
*   **BLAS Level 2**: `fortran_name(dgemv,DGEMV)`, `fortran_name(zgemv,ZGEMV)`
*   **BLAS Level 3**: `fortran_name(dgemm,DGEMM)`, `fortran_name(zgemm,ZGEMM)`, `fortran_name(dsymm,DSYMM)`, `fortran_name(zhemm,ZHEMM)`
*   **LAPACK Solvers**: `fortran_name(dgetrf,DGETRF)` (LU decomp), `fortran_name(dgetrs,DGETRS)` (solve using LU), `fortran_name(zgetri,ZGETRI)` (matrix inverse), `fortran_name(dsysv,DSYSV)` (symmetric system solve), `fortran_name(zhetrf,ZHETRF)` (Hermitian decomp), `fortran_name(zhetrs,ZHETRS)` (solve using Hermitian decomp)
*   **LAPACK Eigenvalue/SVD**: `fortran_name(dgeev,DGEEV)` (general matrix eigenvalues/vectors), `fortran_name(dsyev,DSYEV)` (symmetric eigenvalues/vectors), `fortran_name(zheev,ZHEEV)` (Hermitian eigenvalues/vectors), `fortran_name(dggev,DGGEV)` (generalized eigenvalue problem), `fortran_name(zhegvx,ZHEGVX)` (generalized Hermitian definite eigenproblem), `fortran_name(dgesdd,DGESDD)` (SVD)
*   **LAPACK Utility**: `fortran_name(dlacpy,DLACPY)` (matrix copy), `fortran_name(dlamch,DLAMCH)` (machine constants)

#### C++ Wrapper Functions
*(Corresponding to the Fortran declarations, e.g.,)*
*   `inline void c_dcopy(int n, double *dx, int incx, double *dy, int incy)`
*   `inline void c_zgemm(char transa, char transb, int m, int n, int k, CPX alpha, CPX *a, int lda, CPX *b, int ldb, CPX beta, CPX *c, int ldc)`
*   `inline void c_dgetrf(int m, int n, double *a, int lda, int *ipiv, int *info)`
*   `inline void c_zheev(char jobvl, char uplo, int n, CPX *a, int lda, double *w, CPX *work, int lwork, double *rwork, int *info)`
*   `inline void c_icopy(int n,int *dx,int incx,int *dy,int incy)`: Custom integer copy.

#### C++ Template Functions
*   `template <typename T,typename W> inline void c_tcopy(int n,T *dx,int incx,W *dy,int incy)`: Templated matrix copy.
*   `template <typename T,typename W> inline void c_taxpy(int n, T alpha, T *x, int incx, W *y, int incy)`: Templated AXPY operation.
*   `template <typename T> inline void c_tscal(int n,T da,T *dx,int incx)`: Templated vector scaling.
*   `template <typename T> inline void c_tgemm(...)`: Templated general matrix multiplication.
*   Conditional GPU templates (if `HAVE_SPLITSOLVE` is defined):
    *   `template <typename T> inline void tgemm_dev(...)`: GPU matrix multiplication.
    *   `template <typename T> inline void tgetrf_dev(...)`: GPU LU decomposition.
    *   `template <typename T> inline void tgetrs_dev(...)`: GPU solve using LU.
    *   `template <typename T> inline void tgesv_dev(...)`: GPU general system solve.
    *   `template <typename T> inline void tgetri_dev(...)`: GPU matrix inversion.
    *   `template <typename T> inline void copy_csr_to_device(...)`: Copies CSR matrix to GPU.
    *   `template <typename T> inline void init_var_on_dev(...)`: Initializes variable on GPU.
    *   `template <typename T> inline void init_eye_on_dev(...)`: Initializes identity matrix on GPU.
    *   `template <typename T> inline void csr_mult_f(...)`: CSR matrix multiplication on GPU.
    *   `template <typename T> inline void transpose_matrix(...)`: Matrix transpose on GPU.

### Classes/Structs
*   *None defined in this file, other than those from included headers like `Types.H`.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Example for using a BLAS wrapper (daxpy: y = a*x + y)
// double a = 2.0;
// double x_vec[5] = {1, 2, 3, 4, 5};
// double y_vec[5] = {5, 4, 3, 2, 1};
// int n_elements = 5;
// int increment = 1;
// MYBLAS::c_daxpy(n_elements, a, x_vec, increment, y_vec, increment);

// Example for a LAPACK wrapper (dgetrf: LU decomposition)
// double A_matrix[9] = {2, 1, 1, 4, 3, 3, 8, 7, 9}; // 3x3 matrix, column major
// int n_dim = 3;
// int ipiv_lu[3];
// int info_lu;
// MYBLAS::c_dgetrf(n_dim, n_dim, A_matrix, n_dim, ipiv_lu, &info_lu);
```

## Dependencies and Interactions

### Included Files
*   `#include "mpi.h"`
*   `#include <iostream>`
*   `#include "Types.H"`
*   Conditional includes (if `HAVE_SPLITSOLVE` is defined):
    *   `#include "CWC_utility.H"`
    *   `#include "magma.h"`

### External Libraries
*   **BLAS**: This file extensively wraps BLAS routines.
*   **LAPACK**: This file extensively wraps LAPACK routines.
*   **MPI**: Included (`mpi.h`), though most BLAS/LAPACK routines are serial. MPI might be used by higher-level logic that calls these.
*   **MAGMA**: (If `HAVE_SPLITSOLVE` is defined) Used for GPU-accelerated linear algebra.
*   **CUDA**: (If `HAVE_SPLITSOLVE` is defined) Implied by `CWC_utility.H` for custom GPU kernels.

### Interactions with Other Project Components
*   Provides a vast suite of low-level linear algebra operations that are likely fundamental to many computational modules within OMEN (e.g., solvers, Hamiltonian construction).
*   Depends on `Types.H` for the `CPX` (complex double) type definition.
*   If `HAVE_SPLITSOLVE` is defined, it interacts with GPU utility code from `CWC_utility.H`.

---
*This documentation is auto-generated and may require further manual refinement.*
