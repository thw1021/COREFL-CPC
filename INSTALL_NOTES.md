# COREFL-CPC Installation Notes

## Environment

| Item | Detail |
|------|--------|
| OS | Ubuntu 22.04.5 LTS (x86_64) |
| CPU | 13th Gen Intel Core i5-13500H |
| GPU | NVIDIA GeForce RTX 3050 4GB Laptop GPU |
| Compute Capability | 8.6 |
| Driver | 535.309.01 |
| CUDA | 12.2 (`/usr/local/cuda-12.2`) |
| GCC | 11.4.0 |
| OpenMPI | 4.1.2 (system apt, `/usr/bin/mpicxx.openmpi`) |
| CMake | 3.30.5 |
| Build Type | Release |

## Pre-build Fix

`src/DParameter.cuh` had sponge-layer members commented out, causing `WENO.cu` to fail.  
Added missing fields: `x_sponge_start`, `y_sponge_start`, and uncommented all other sponge-layer members.

## Build Commands

```bash
# 1. Load environment
source ~/.bashrc

# 2. Clean & configure
rm -rf build && mkdir build
cmake -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_ARCHITECTURES=86 \
  -DMPI_CXX_COMPILER=/usr/bin/mpicxx.openmpi

# 3. Build with 10 parallel jobs
cmake --build build --parallel 10
```

## Important Notes

- **MPI**: conda's `mpicxx` (`~/miniforge3/bin/mpicxx`) is broken (missing `x86_64-conda-linux-gnu-c++`). Always pass `-DMPI_CXX_COMPILER=/usr/bin/mpicxx.openmpi` to use the system OpenMPI.
- **CUDA_ARCHITECTURES**: set to `86` for this GPU (compute capability 8.6). For other GPUs, check with `nvidia-smi --query-gpu=compute_cap --format=csv,noheader` and use the value without the dot (e.g., 70 for V100, 80 for A100).
- **CMakeLists.txt defaults** (set in project root `CMakeLists.txt`):
  - `CMAKE_CUDA_ARCHITECTURES=86`
  - `MAX_SPEC_NUMBER=9`
  - `MAX_REAC_NUMBER=20`
  - `MAX_PASSIVE_SCALAR_NUMBER=1`
  - `HighTempMultiPart` (for high-temperature air chemistry; use `Combustion2Part` for general combustion)
- The only compiler warning (harmless) is in `src/gxl_lib/MyString.cpp:88`: ignoring `fread` return value.

## Output

```
/home/tang/packages/COREFL-CPC/corefl    # 28 MB, ELF 64-bit executable
```

## Quick Run

```bash
mpirun -n 1 /home/tang/packages/COREFL-CPC/corefl
```
