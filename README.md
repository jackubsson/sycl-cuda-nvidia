# OneApiCuda — Enabling NVIDIA GPU Execution Through Intel SYCL

**Objective:** Build and validate a custom Intel LLVM/SYCL toolchain with NVIDIA CUDA backend support on Windows.

**Work performed:**
- Configured a Windows x64 development environment using Visual Studio Build Tools, Python, CMake, and the NVIDIA CUDA environment.
- Cloned and configured Intel LLVM's SYCL branch with CUDA backend support.
- Built and deployed the custom SYCL toolchain, including the Clang compiler, Unified Runtime CUDA adapter, and NVIDIA device libraries.
- Diagnosed runtime device-discovery and DLL-loading issues.
- Used SYCL device enumeration to verify that the CUDA backend detected the NVIDIA GeForce RTX 3050 Ti Laptop GPU.
- Compiled and executed a SYCL kernel targeting `nvptx64-nvidia-cuda` and verified the expected output and successful process exit.

**Result:** Successfully demonstrated end-to-end SYCL compilation and kernel execution on NVIDIA hardware using a custom-built Intel LLVM/SYCL CUDA-enabled toolchain.

**Technologies:** C++, SYCL, Intel LLVM/Clang, Unified Runtime, NVIDIA CUDA, CMake, Python, Visual Studio Build Tools, PowerShell, Windows x64.
