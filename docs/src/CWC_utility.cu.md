# Documentation for `src/CWC_utility.cu`

## Overview

This CUDA source file (`.cu`) provides a collection of utility functions to support GPU-accelerated computations within the OMEN project. These utilities include:
*   Initialization and finalization wrappers for CUDA libraries like cuBLAS and cuSPARSE.
*   GPU device management (setting the active GPU).
*   GPU memory management (allocation, deallocation, data transfers between host and device).
*   Wrappers for specific cuBLAS operations (e.g., `dgemm`, `zgemm`, `zaxpy`).
*   Custom CUDA kernels for various array and matrix operations, such as initializing data, extracting matrix sub-blocks (diagonal/off-diagonal), transposing matrices, and symmetrizing matrices.
*   A simple mechanism for tracking allocated CUDA memory.

These functions are primarily C-style `extern "C"` functions, making them callable from C++ code, and CUDA global kernels for on-device execution.

## Key Components

### Functions (`extern "C"`)
*   `void set_gpu(int dev, char *gpu_string)`: Sets the active CUDA device and retrieves its name.
*   `void cublas_init(void **handle)`: Initializes cuBLAS and creates a handle.
*   `void cublas_finalize(void *handle)`: Finalizes cuBLAS and destroys the handle.
*   `void cusparse_init(void **handle)`: Initializes cuSPARSE and creates a handle.
*   `void cusparse_finalize(void *handle)`: Finalizes cuSPARSE and destroys the handle.
*   `size_t allocate_data_on_device(void **data, size_t size_data)`: Allocates memory on the GPU.
*   `void deallocate_data_on_device(void *data)`: Frees GPU memory (older version, does not update `c_memory`).
*   `size_t deallocate_data_on_dev(void *data, size_t size_data)`: Frees GPU memory and updates the internal memory counter.
*   `void copy_data_to_device(void *host_data, void *device_data, int N, int M, size_t size_element)`: Copies a 2D matrix from host to device using `cublasSetMatrixAsync`.
*   `void memcpy_to_device(void *host_data, void *device_data, size_t size_element)`: Copies data from host to device using `cudaMemcpyAsync`.
*   `void copy_data_to_host(void *host_data, void *device_data, int N, int M, size_t size_element)`: Copies a 2D matrix from device to host using `cublasGetMatrixAsync`.
*   `void memcpy_to_host(void *host_data, void *device_data, size_t size_element)`: Copies data from device to host using `cudaMemcpyAsync`.
*   `void dgemm_on_dev(...)`: Wrapper for `cublasDgemm` (double-precision matrix-matrix multiplication).
*   `void zgemm_on_dev(...)`: Wrapper for `cublasZgemm` (complex double-precision matrix-matrix multiplication).
*   `void zaxpy_on_dev(...)`: Wrapper for `cublasZaxpy` (complex vector addition: y = alpha*x + y).
*   `void d_init_var_on_dev(double *var, int N, cudaStream_t stream)`: Launches kernel to initialize a double array on GPU to zeros.
*   `void d_init_eye_on_dev(double *var, int N, cudaStream_t stream)`: Launches kernel to initialize a double square matrix on GPU to identity.
*   `void z_init_var_on_dev(CPX *var, int N, cudaStream_t stream)`: Launches kernel to initialize a CPX array on GPU to zeros.
*   `void z_init_eye_on_dev(CPX *var, int N, cudaStream_t stream)`: Launches kernel to initialize a CPX square matrix on GPU to identity.
*   `void correct_diag_on_dev(CPX *var, int N, cudaStream_t stream)`: Launches kernel to set imaginary part of diagonal elements to zero.
*   `void change_var_type_on_dev(double *var1, CPX *var2, int N, cudaStream_t stream)`: Launches kernel to convert a double array to a CPX array (imaginary part is zero).
*   `void change_sign_imag_on_dev(CPX *var, int N)`: Launches kernel to negate the imaginary part of a CPX array.
*   `void d_extract_diag_on_dev(...)`: Launches kernel to extract diagonal block from a sparse matrix (double precision from complex nnz).
*   `void d_extract_not_diag_on_dev(...)`: Launches kernel to extract an off-diagonal block (double precision from complex nnz).
*   `void z_extract_diag_on_dev(...)`: Launches kernel to extract diagonal block (complex precision).
*   `void z_extract_not_diag_on_dev(...)`: Launches kernel to extract an off-diagonal block (complex precision).
*   `void d_copy_csr_to_device(...)`: Copies CSR matrix components (double nnz) from host to device.
*   `void z_copy_csr_to_device(...)`: Copies CSR matrix components (CPX nnz) from host to device.
*   `void d_csr_mult_f(...)`: Wrapper for `cusparseDcsrmm` (CSR sparse matrix-dense matrix multiplication, double).
*   `void z_csr_mult_f(...)`: Wrapper for `cusparseZcsrmm` (CSR sparse matrix-dense matrix multiplication, complex).
*   `void z_csr_mult_fo(...)`: Similar to `z_csr_mult_f` but with `CUSPARSE_INDEX_BASE_ZERO`.
*   `void d_transpose_matrix(...)`: Launches kernel to transpose a double matrix.
*   `void z_transpose_matrix(...)`: Launches kernel to transpose a complex matrix (conjugate transpose).
*   `void d_symmetrize_matrix(...)`: Launches kernel to symmetrize a double matrix.
*   `void z_symmetrize_matrix(...)`: Launches kernel to make a complex matrix Hermitian.
*   `void z_symmetrize_matrix_2(...)`: Launches kernel for another form of symmetrization/anti-symmetrization.

### CUDA Kernels (`__global__`)
*   `d_init_variable_on_dev`: Initializes elements of a double array to 0.0.
*   `d_init_eye_on_device`: Initializes a square double matrix to an identity matrix.
*   `z_init_variable_on_dev`: Initializes elements of a `cuDoubleComplex` array to (0.0, 0.0).
*   `z_init_eye_on_device`: Initializes a square `cuDoubleComplex` matrix to an identity matrix.
*   `correct_diag_on_device`: Sets the imaginary part of diagonal elements of a `cuDoubleComplex` matrix to 0.0.
*   `change_variable_type_on_dev`: Converts elements from `double` to `cuDoubleComplex` (imaginary part set to 0.0).
*   `change_sign_imaginary_part_on_dev`: Negates the imaginary part of `cuDoubleComplex` array elements.
*   `d_extract_diag`: Extracts diagonal block elements from a CSR format (complex `nnz` to double `D`).
*   `d_extract_not_diag`: Extracts off-diagonal block elements from a CSR format (complex `nnz` to double `D`).
*   `z_extract_diag`: Extracts diagonal block elements from a CSR format (complex `nnz` to complex `D`).
*   `z_extract_not_diag`: Extracts off-diagonal block elements from a CSR format (complex `nnz` to complex `D`).
*   `d_transpose`: Transposes a 2D array of doubles.
*   `z_transpose`: Transposes a 2D array of `cuDoubleComplex`, performing a conjugate transpose.
*   `d_symmetrize`: Symmetrizes a double matrix: `M_ij = (M_ij + M_ji) / 2`.
*   `z_symmetrize`: Makes a complex matrix Hermitian: `M_ij = (M_ij + conj(M_ji)) / 2`.
*   `z_symmetrize_2`: Another symmetrization variant, setting diagonal to `(0, imag(M_ii))` and `M_ji = (-real(M_ij), imag(M_ij))`.

### Macros
*   `max_stream`: Default value 16, likely for maximum number of CUDA streams (though not explicitly used in this snippet).
*   `BLOCK_DIM`: Default value 16, defining the CUDA thread block dimension for kernels.

## Important Variables/Constants
*   `static volatile size_t c_memory = 0;`: A static variable used to track the total amount of memory allocated on the GPU by these utilities.

## Usage Examples

```c++
// Conceptual example of using a utility function
// void *d_my_array;
// size_t array_size = 100 * sizeof(double);
// allocate_data_on_device(&d_my_array, array_size);
// // ... use d_my_array ...
// deallocate_data_on_dev(d_my_array, array_size);

// cublasHandle_t blas_handle;
// cublas_init((void**)&blas_handle);
// // ... use blas_handle for cuBLAS calls via wrappers ...
// cublas_finalize(blas_handle);
```

## Dependencies and Interactions

### Included Files
*   `#include <stdio.h>`
*   `#include "Types.H"`
*   `#include "cublas_v2.h"`
*   `#include "cusparse_v2.h"`
*   `#include "cuda.h"` (Likely refers to `cuda_runtime_api.h` or similar core CUDA headers)

### External Libraries
*   **CUDA Toolkit**: Essential for all functions and kernels in this file.
    *   **CUDA Runtime API**: Used for memory management (`cudaMalloc`, `cudaFree`, `cudaMemcpyAsync`), device properties (`cudaGetDeviceProperties`), and kernel launches (`<<<...>>>`).
    *   **cuBLAS**: Used for BLAS operations on the GPU (e.g., `cublasCreate`, `cublasDgemm`, `cublasZgemm`).
    *   **cuSPARSE**: Used for sparse matrix operations on the GPU (e.g., `cusparseCreate`, `cusparseZcsrmm`).

### Interactions with Other Project Components
*   This file provides low-level GPU utilities that are likely used by higher-level computational modules within OMEN, such as solvers (e.g., `SplitSolve.H` includes `CWC_utility.H`).
*   It depends on `Types.H` for the `CPX` (complex double-precision) type definition, which is mapped to `cuDoubleComplex` for CUDA operations.

---
*This documentation is auto-generated and may require further manual refinement.*
