# Documentation for `src/rGF.C`

## Overview

This file implements the `rGF` class, which provides a solver based on the recursive Green's function (rGF) technique. This method is commonly used in condensed matter physics and quantum transport to calculate the Green's function of a system, particularly for systems that can be partitioned into a series of blocks (e.g., layered devices). The rGF algorithm computes the Green's function of the system by recursively incorporating these blocks.

The implementation includes methods for extracting diagonal and off-diagonal blocks from a larger sparse matrix, calculating intermediate "right-looking" Green's functions (g<sup>R</sup>) and self-energies (&Sigma;<sup>R</sup>), and finally assembling the desired blocks of the full Green's function (G<sup>R</sup>).

## Key Components

### Functions
*(These are member functions of the `rGF` class, declared in `rGF.H`)*
*   `rGF::rGF(TCSR<CPX>* e_minus_h)`:
    *   **Description**: Constructor for the `rGF` solver.
    *   **Parameters**:
        *   `e_minus_h`: A pointer to a `TCSR<CPX>` matrix representing (E*I - H), where E is energy, I is identity, and H is the Hamiltonian.
*   `rGF::~rGF()`:
    *   **Description**: Destructor.
*   `void rGF::solve_blocks(std::vector<int> Bmin, std::vector<int> Bmax, CPX *GR, CPX *GRNNp1)`:
    *   **Description**: The main rGF algorithm to solve for and store diagonal blocks (`GR`) and the first upper off-diagonal blocks (`GRNNp1`) of the Green's function. It involves a forward recursion to calculate g<sup>R</sup> matrices and a backward recursion to calculate G<sup>R</sup> blocks.
    *   **Parameters**:
        *   `Bmin`: Vector of start row/column indices for each block.
        *   `Bmax`: Vector of end row/column indices for each block.
        *   `GR`: (Output) Pointer to store concatenated diagonal blocks of G<sup>R</sup>.
        *   `GRNNp1`: (Output) Pointer to store concatenated G<sup>R</sup><sub>i,i+1</sub> blocks.
*   `void rGF::get_diagonal_block(int block, std::vector<int> Bmin, std::vector<int> Bmax, CPX *diagonal_block)`:
    *   **Description**: Extracts a specified diagonal block from the main system matrix (`this->matrix`) and stores it as a dense matrix in `diagonal_block`.
*   `void rGF::get_Tip1i_csc_Tiip1_csr(int block, std::vector<int> Bmin, std::vector<int> Bmax, TCSC<CPX,int> *T_ip1i, TCSR<CPX> *T_iip1)`:
    *   **Description**: Extracts the off-diagonal blocks T<sub>i+1,i</sub> (into `T_ip1i` as CSC) and T<sub>i,i+1</sub> (into `T_iip1` as CSR) from the main matrix. Assumes T<sub>i,i+1</sub> is derived from T<sub>i+1,i</sub>.
*   `void rGF::get_Tip1i_csr_Tiip1_csc(int block, std::vector<int> Bmin, std::vector<int> Bmax, TCSR<CPX> *T_ip1i, TCSC<CPX,int> *T_iip1)`:
    *   **Description**: Extracts the off-diagonal blocks T<sub>i+1,i</sub> (into `T_ip1i` as CSR) and T<sub>i,i+1</sub> (into `T_iip1` as CSC) from the main matrix. Assumes T<sub>i+1,i</sub> is derived from T<sub>i,i+1</sub>.
*   `void rGF::calculate_gR_rec(int block, std::vector<int> Bmin, std::vector<int> Bmax, std::vector<int> GR_start_index, CPX *gR)`:
    *   **Description**: Recursively calculates a block of the right-looking Green's function g<sup>R</sup><sub>i</sub> = ( (E-H)<sub>ii</sub> - &Sigma;<sup>R</sup><sub>i</sub> )<sup>-1</sup>. &Sigma;<sup>R</sup><sub>i</sub> (calculated by `calculate_sigmaR`) is assumed to be in `tmp0`.
*   `void rGF::calculate_gR_init(int block, std::vector<int> Bmin, std::vector<int> Bmax, std::vector<int> GR_start_index, CPX *gR)`:
    *   **Description**: Initializes the g<sup>R</sup> calculation for the last block: g<sup>R</sup><sub>N</sub> = ( (E-H)<sub>NN</sub> )<sup>-1</sup> (assuming self-energy for the last block is zero or already included in H).
*   `void rGF::calculate_sigmaR(int block, std::vector<int> Bmin, std::vector<int> Bmax, std::vector<int> GR_start_index, CPX *gR)`:
    *   **Description**: Calculates the self-energy &Sigma;<sup>R</sup><sub>i</sub> = T<sub>i,i+1</sub> * g<sup>R</sup><sub>i+1</sub> * T<sub>i+1,i</sub>. The result is stored in the private member `tmp0`.
*   `void rGF::calculate_GR(int block, std::vector<int> Bmin, std::vector<int> Bmax, CPX *gR_i, CPX *GR_ii, CPX *GR_im1i)`:
    *   **Description**: Calculates a diagonal block G<sup>R</sup><sub>i,i</sub> and an off-diagonal block G<sup>R</sup><sub>i-1,i</sub> of the full Green's function using the previously computed g<sup>R</sup> and G<sup>R</sup><sub>i-1,i-1</sub> blocks.
    *   G<sup>R</sup><sub>i,i</sub> = g<sup>R</sup><sub>i</sub> + g<sup>R</sup><sub>i</sub> * T<sub>i,i-1</sub> * G<sup>R</sup><sub>i-1,i-1</sub> * T<sub>i-1,i</sub> * g<sup>R</sup><sub>i</sub>
    *   G<sup>R</sup><sub>i-1,i</sub> = - G<sup>R</sup><sub>i-1,i-1</sub> * T<sub>i-1,i</sub> * g<sup>R</sup><sub>i</sub>
*   `void rGF::transpose_full_matrix(CPX *normal, int rows, int cols, CPX *transposed)`:
    *   **Description**: Transposes a dense matrix stored in column-major (Fortran-style) order.
*   `void rGF::set_to_unity(int diagonal_length, CPX *unity)`:
    *   **Description**: Sets a dense square matrix `unity` to the identity matrix.

### Classes/Structs
*   *None defined in this file. It implements methods for the `rGF` class declared in `rGF.H`.*

## Important Variables/Constants
*   *None defined globally in this file. Class members like `matrix`, `sparse_CSR`, `sparse_CSC`, `tmp0`, `tmp1`, `tmp2` are used internally.*

## Usage Examples

```c++
// Conceptual example for using the rGF solver
// TCSR<CPX> system_matrix(...); // Initialized E*I - H matrix
// std::vector<int> block_min_indices, block_max_indices;
// // ... populate block_min_indices and block_max_indices ...
// int total_GR_size = ...; // Calculate total size needed for GR blocks
// int total_GRNNp1_size = ...; // Calculate total size for GRNNp1 blocks
// CPX* GR_results = new CPX[total_GR_size];
// CPX* GRNNp1_results = new CPX[total_GRNNp1_size];

// rGF solver(&system_matrix);
// solver.solve_blocks(block_min_indices, block_max_indices, GR_results, GRNNp1_results);

// // Results are in GR_results and GRNNp1_results
// delete[] GR_results;
// delete[] GRNNp1_results;
```

## Dependencies and Interactions

### Included Files
*   `#include <vector>`
*   `#include <assert.h>`
*   `#include <string.h>`
*   `#include <sstream>`
*   `#include <algorithm>`
*   `#include "rGF.H"`

### External Libraries
*   **BLAS/LAPACK**: Implied by the use of functions like `c_zcopy`, `c_zaxpy`, `c_zgemm` (for matrix multiplications) and `c_zgetrf`, `c_zgetrs` (for LU decomposition and solve, used for matrix inversion). These are likely wrappers defined in `Blas.H` (included via `rGF.H` or `CSR.H`).

### Interactions with Other Project Components
*   This file implements the `rGF` class, which is declared in `rGF.H`.
*   It uses `TCSR<CPX>` (from `CSR.H`) and `TCSC<CPX,int>` (from `CSC.H`) for sparse matrix representations.
*   The `rGF` solver is a core numerical method that can be used by other parts of the OMEN project, for example, by the `tmprGF::sparse_invert` function for inverting matrices, or directly for calculating Green's function blocks in transport simulations.

---
*This documentation is auto-generated and may require further manual refinement.*
