# Documentation for `src/CWC_utility.H`

## Overview

This header file declares a set of C-style functions that provide an interface to CUDA (Compute Unified Device Architecture) utilities. These functions are implemented in the corresponding CUDA source file (`CWC_utility.cu`) and are designed to be callable from C++ code to leverage GPU acceleration for various computational tasks within the OMEN project.

The declared utilities cover:
*   GPU device selection and information.
*   Initialization and finalization of CUDA libraries like cuBLAS and cuSPARSE.
*   GPU memory management (allocation, deallocation, host-device transfers).
*   Wrappers for specific cuBLAS functions (e.g., matrix multiplication).
*   Wrappers for custom CUDA kernels (e.g., for matrix initialization, extraction, transposition, symmetrization).

## Key Components

### Functions
*(All functions are declared `extern "C"`)*
*   `void set_gpu(int dev, char* gpu_string)`: Sets the active CUDA device and gets its name.
*   `void cublas_init(void** handle)`: Initializes cuBLAS.
*   `void cublas_finalize(void* handle)`: Finalizes cuBLAS.
*   `void cusparse_init(void** handle)`: Initializes cuSPARSE.
*   `void cusparse_finalize(void* handle)`: Finalizes cuSPARSE.
*   `size_t allocate_data_on_device(void** data, size_t size_data)`: Allocates memory on the GPU.
*   `void deallocate_data_on_device(void* data)`: Frees GPU memory (older version).
*   `size_t deallocate_data_on_dev(void* data, size_t size_data)`: Frees GPU memory and updates internal counter.
*   `void copy_data_to_device(void* host_data, void* device_data, int N, int M, size_t size_element)`: Copies a 2D matrix from host to device.
*   `void memcpy_to_device(void* host_data, void* device_data, size_t size_element)`: Copies data from host to device.
*   `void copy_data_to_host(void* host_data, void* device_data, int N, int M, size_t size_element)`: Copies a 2D matrix from device to host.
*   `void memcpy_to_host(void* host_data, void* device_data, size_t size_element)`: Copies data from device to host.
*   `void dgemm_on_dev(...)`: Wrapper for double-precision matrix-matrix multiplication on GPU.
*   `void zgemm_on_dev(...)`: Wrapper for complex double-precision matrix-matrix multiplication on GPU.
*   `void zaxpy_on_dev(...)`: Wrapper for complex vector addition (y = alpha*x + y) on GPU.
*   `void d_copy_csr_to_device(...)`: Copies a CSR matrix (double nnz) to GPU.
*   `void z_copy_csr_to_device(...)`: Copies a CSR matrix (CPX nnz) to GPU.
*   `void d_csr_mult_f(...)`: Wrapper for CSR sparse matrix-dense matrix multiplication (double) on GPU.
*   `void z_csr_mult_f(...)`: Wrapper for CSR sparse matrix-dense matrix multiplication (complex) on GPU.
*   `void z_csr_mult_fo(...)`: Similar to `z_csr_mult_f` with different indexing base for cuSPARSE.
*   `void d_transpose_matrix(double* odata, double* idata, int size_x, int size_y)`: Transposes a double matrix on GPU.
*   `void z_transpose_matrix(CPX* odata, CPX* idata, int size_x, int size_y)`: Transposes a complex matrix (conjugate transpose) on GPU.
*   `void d_init_var_on_dev(double* var, int N, cudaStream_t stream)`: Initializes a double array to zeros on GPU.
*   `void d_init_eye_on_dev(double* var, int N, cudaStream_t stream)`: Initializes a double matrix to identity on GPU.
*   `void z_init_var_on_dev(CPX* var, int N, cudaStream_t stream)`: Initializes a CPX array to zeros on GPU.
*   `void z_init_eye_on_dev(CPX* var, int N, cudaStream_t stream)`: Initializes a CPX matrix to identity on GPU.
*   `void correct_diag_on_dev(CPX* var, int N, cudaStream_t stream)`: Sets imaginary part of diagonal elements to zero on GPU.
*   `void change_var_type_on_dev(double* var1, CPX* var2, int N, cudaStream_t stream)`: Converts a double array to CPX array on GPU.
*   `void change_sign_imag_on_dev(CPX* var, int N)`: Negates imaginary part of a CPX array on GPU.
*   `void d_extract_diag_on_dev(...)`: Extracts diagonal block from CSR matrix (double from CPX nnz) on GPU.
*   `void d_extract_not_diag_on_dev(...)`: Extracts off-diagonal block from CSR matrix (double from CPX nnz) on GPU.
*   `void d_symmetrize_matrix(double* matrix, int N, cudaStream_t stream)`: Symmetrizes a double matrix on GPU.
*   `void z_extract_diag_on_dev(...)`: Extracts diagonal block from CSR matrix (CPX) on GPU.
*   `void z_extract_not_diag_on_dev(...)`: Extracts off-diagonal block from CSR matrix (CPX) on GPU.
*   `void z_symmetrize_matrix(CPX* matrix, int N, cudaStream_t stream)`: Makes a complex matrix Hermitian on GPU.
*   `void z_symmetrize_matrix_2(CPX* matrix, int N, cudaStream_t stream)`: Another variant of complex matrix symmetrization on GPU.

### Classes/Structs
*   *None defined in this file.*

## Important Variables/Constants
*   *None defined globally in this file.*

## Usage Examples

```c++
// Conceptual example:
// char gpu_name[256];
// set_gpu(0, gpu_name); // Select GPU 0
// printf("Using GPU: %s\n", gpu_name);

// void* d_matrix_gpu;
// size_t matrix_bytes = 1000 * sizeof(double);
// allocate_data_on_device(&d_matrix_gpu, matrix_bytes);
// // ... use d_matrix_gpu ...
// deallocate_data_on_dev(d_matrix_gpu, matrix_bytes);
```

## Dependencies and Interactions

### Included Files
*   `#include <cuda_runtime.h>`
*   `#include <stdio.h>`
*   `#include "Types.H"`

### External Libraries
*   **CUDA Toolkit (Runtime API)**: Indicated by `<cuda_runtime.h>`. These declarations are for functions that will internally call CUDA API functions.

### Interactions with Other Project Components
*   This header file declares the C-style interface for CUDA utility functions that are implemented in `CWC_utility.cu`.
*   Modules that require GPU acceleration (e.g., potentially `SplitSolve.H`) will include this header to call these GPU utility functions.
*   Depends on `Types.H` for the `CPX` (complex double-precision) type definition.

---
*This documentation is auto-generated and may require further manual refinement.*
