# Documentation for `src/rGF.H`

## Overview

The existing comment "M_TODO: write documentation" indicates that a detailed overview needs to be written for this file. It likely defines the `rGF` class which seems to be related to Green's functions calculations, possibly for recursive methods.

## Key Components

### Classes
*   `rGF`: This class is central to the file. The comment "M_TODO: write documentation" applies to the class as well. It appears to manage matrix operations, possibly related to Green's functions.
    *   **Public Methods**:
        *   `rGF(TCSR<CPX>* mat)`: Constructor, likely initializes the object with a matrix.
        *   `~rGF()`: Destructor.
        *   `solve_blocks(std::vector<int>, std::vector<int>, CPX*, CPX*)`: Solves parts of the system.
    *   **Private Methods**:
        *   `get_diagonal_block(int, std::vector<int>, std::vector<int>, CPX*)`: Extracts a diagonal block from the matrix.
        *   `get_Tip1i_csr_Tiip1_csc(int, std::vector<int>, std::vector<int>, TCSR<CPX>*, TCSC<CPX,int>*)`: Likely retrieves specific matrix blocks in CSR and CSC formats.
        *   `get_Tip1i_csc_Tiip1_csr(int, std::vector<int>, std::vector<int>, TCSC<CPX,int>*, TCSR<CPX>*)`: Similar to the above, possibly with different storage order.
        *   `calculate_gR_rec(int, std::vector<int>, std::vector<int>, std::vector<int>, CPX*)`: Recursively calculates parts of the Green's function.
        *   `calculate_gR_init(int, std::vector<int>, std::vector<int>, std::vector<int>, CPX*)`: Initializes parts of the Green's function calculation.
        *   `calculate_sigmaR(int,  std::vector<int>, std::vector<int>, std::vector<int>, CPX*)`: Calculates a self-energy component (Sigma R).
        *   `calculate_GR(int, std::vector<int>, std::vector<int>, CPX*, CPX*, CPX*)`: Calculates the retarded Green's function (GR).
        *   `transpose_full_matrix(CPX*, int, int, CPX*)`: Transposes a matrix.
        *   `set_to_unity(int, CPX*)`: Sets a matrix to identity or a similar concept.
        *   `test_x(TCSC<CPX,int>*, TCSR<CPX>*, int)`: A test function.

### Functions
*(Functions listed above are members of the `rGF` class)*

## Important Variables/Constants
*Within the `rGF` class (private members):*
*   `matrix`: (Type `TCSR<CPX>*`) Stores the primary matrix, likely `E * 1 - H`.
*   `fortran_index`: (Type `int`) Likely used for 1-based indexing if interacting with Fortran code.
*   `sparse_CSR`: (Type `TCSR<CPX>*`) Working memory for a sparse matrix in CSR format.
*   `sparse_CSC`: (Type `TCSC<CPX,int>*`) Working memory for a sparse matrix in CSC format.
*   `tmp0`, `tmp1`, `tmp2`: (Type `CPX*`) Temporary storage buffers.

## Usage Examples

```c++
// Example for the rGF class (conceptual)
// TCSR<CPX> some_matrix;
// // ... populate some_matrix ...
// rGF green_function_calculator(&some_matrix);
// std::vector<int> blocks_to_solve_row, blocks_to_solve_col;
// CPX* results_ptr;
// CPX* input_ptr;
// // ... initialize blocks_to_solve_row, blocks_to_solve_col, results_ptr, input_ptr ...
// green_function_calculator.solve_blocks(blocks_to_solve_row, blocks_to_solve_col, results_ptr, input_ptr);
```

## Dependencies and Interactions

### Included Files
*   `#include "CSR.H"`
*   `#include "CSC.H"`
*   `#include "Types.H"`
*   `#include "Utilities.H"`

### External Libraries
*No external libraries explicitly identified from includes. Further review may be needed.*

### Interactions with Other Project Components
*Interacts with components defined in `CSR.H`, `CSC.H`, `Types.H`, and `Utilities.H`. The specific nature of these interactions (e.g., data structures used, functions called) would require deeper analysis of the source code. Likely uses sparse matrix formats (CSR/CSC) and complex number types (CPX) defined in these headers.*

---
*This documentation is auto-generated and may require further manual refinement.*
