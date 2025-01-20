---
{"dg-publish":true,"permalink":"/AI/nvidia/cuda/算子开发 - cuda syntax/","noteIcon":"3"}
---

#cuda
## 1.关键词介绍

### 1.1 Declaring functions

Global functions are also called "kernels". It's the functions that you may call from the host side using CUDA kernel call semantics (`<<<...>>>`).

|                   |                                                                  |
| ----------------- | ---------------------------------------------------------------- |
| `__global__`      | declares kernel, which is called on host and executed on device  |
| `__device__`      | declares device function, which is called and executed on device |
| `__host__`        | declares host function, which is called and executed on host     |
| `__noinline__`    | to avoid inlining                                                |
| `__forceinline__` | to force inlining                                                |

### 1.2 Declaring variables

|                |                                                                                                                         |
| -------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `__device__`   | declares device variable in global memory, accessible from all threads, with lifetime of application                    |
| `__constant__` | declares device variable in constant memory, accessible from all threads, with lifetime of application                  |
| `__shared__`   | declares device varibale in `block's shared memory`, accessible from all threads within a block, with lifetime of block |
| `__restrict__` | standard C definition that pointers are not aliased                                                                     |

### 1.3 build-in variables

**Grid and Blocks:** In CUDA, a grid is composed of blocks, and each block contains multiple threads. Both grids and blocks can be organized in 1D, 2D, or 3D layouts.

If you have a 2D grid of 3x2 blocks (gridDim.x = 3, gridDim.y = 2) and each block is 4x4 threads (blockDim.x = 4, blockDim.y = 4), then:

• blockIdx.x could be 0, 1, or 2, representing the horizontal position of the block.
• blockIdx.y could be 0 or 1, representing the vertical position of the block.
• threadIdx.x could be 0, 1, 2, or 3, representing the position within the block horizontally.
• threadIdx.y could be 0, 1, 2, or 3, representing the position within the block vertically.
  
This combination allows each thread in the grid to have a unique identifier, which is crucial for parallel processing on the GPU.

|             |                                                                       |
| ----------- | --------------------------------------------------------------------- |
| `blockIdx`  | represents the index of the current block in a grid of thread blocks. |
| `threadIdx` | threadIdx specifies the index of the thread within its block.         |
|             |                                                                       |
|             |                                                                       |



### references:
https://icl.utk.edu/~mgates3/docs/cuda.html
https://stackoverflow.com/questions/12373940/difference-between-global-and-device-functions