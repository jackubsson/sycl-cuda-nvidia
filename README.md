# OneApiCuda — SYCL on NVIDIA GPUs for Windows

Build and validate a custom Intel LLVM/SYCL toolchain with NVIDIA CUDA
backend support on Windows x64.

The goal of this project is to enable SYCL C++ applications to execute
kernels on NVIDIA GPUs through a CUDA backend, while retaining the
SYCL programming model.

## Project Status

**Proof of concept: NVIDIA SYCL kernel execution verified.**

The development environment successfully discovered an NVIDIA GPU using
the custom SYCL runtime and executed a test kernel on the GPU.

Verified hardware:

- NVIDIA GeForce RTX 3050 Ti Laptop GPU
- NVIDIA compute capability: 8.6 (`sm_86`)
- Intel UHD Graphics integrated GPU
- Windows x64

## Motivation

SYCL provides a single C++ programming model for heterogeneous computing.
This project explores a custom Intel LLVM/SYCL toolchain configured to
target NVIDIA GPUs through its CUDA backend.

The main challenge was not ordinary CUDA compilation. It was enabling
SYCL device discovery, compilation, runtime dispatch, and kernel execution
through the custom SYCL CUDA backend.

## Architecture

```text
SYCL C++ application
        |
        v
Custom Intel LLVM / Clang compiler
        |
        | -fsycl
        | -fsycl-targets=nvptx64-nvidia-cuda
        v
NVIDIA PTX device code
        |
        v
SYCL runtime and Unified Runtime CUDA adapter
        |
        v
NVIDIA CUDA runtime and driver
        |
        v
GeForce RTX 3050 Ti Laptop GPU
```

## Technology Stack

- C++
- SYCL
- Intel LLVM / Clang
- Unified Runtime
- NVIDIA CUDA
- CMake
- Python
- Visual Studio Build Tools
- Windows PowerShell

## Development Environment

The original development environment used:

- Windows x64
- CUDA Toolkit 12.9
- NVIDIA RTX 3050 Ti Laptop GPU
- Compute architecture `sm_86`
- A custom-built Intel LLVM/SYCL toolchain

The CUDA Toolkit version and the CUDA version reported by the NVIDIA
driver are different concepts. Use the installed toolkit and driver
versions appropriate for your environment.

## Build Configuration

Example PowerShell configuration:

```powershell
$LLVM = "C:\Users\wqads\Documents\OneApiCuda\sycl-cuda-clean\build\install"
$CUDA_PATH = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.9"

$env:PATH = "$LLVM\bin;$CUDA_PATH\bin;$env:PATH"
```

Check the available SYCL devices:

```powershell
& "$LLVM\bin\sycl-ls.exe"
```

The tested environment listed devices similar to:

```text
[level_zero:gpu] Intel(R) UHD Graphics
[opencl:gpu] Intel(R) UHD Graphics
[opencl:cpu] 11th Gen Intel(R) Core(TM) i5-11400H
[cuda:gpu] NVIDIA GeForce RTX 3050 Ti Laptop GPU 8.6
```

The precise device list depends on the installed drivers, runtime
adapters, and build configuration.

## Compile a SYCL CUDA Application

Example command for an existing test source named `sycl_test.cpp`:

```powershell
$env:ONEAPI_DEVICE_SELECTOR = "cuda:gpu"

& "$LLVM\bin\clang++.exe" `
    -fsycl `
    -fsycl-targets=nvptx64-nvidia-cuda `
    -Xsycl-target-backend=nvptx64-nvidia-cuda `
    --offload-arch=sm_86 `
    "--cuda-path=$CUDA_PATH" `
    sycl_test.cpp `
    -o sycl_test.exe

if ($LASTEXITCODE -ne 0) {
    throw "SYCL compilation failed."
}

.\sycl_test.exe

if ($LASTEXITCODE -ne 0) {
    throw "SYCL test failed."
}
```

Run these commands in an environment where the required compiler,
runtime DLLs, CUDA libraries, and Visual Studio dependencies are available.

## Device Selection

Select the NVIDIA CUDA backend:

```powershell
$env:ONEAPI_DEVICE_SELECTOR = "cuda:gpu"
.\sycl_test.exe
```

Select the Intel Level Zero GPU backend:

```powershell
$env:ONEAPI_DEVICE_SELECTOR = "level_zero:gpu"
.\sycl_test.exe
```

Select the Intel OpenCL CPU backend:

```powershell
$env:ONEAPI_DEVICE_SELECTOR = "opencl:cpu"
.\sycl_test.exe
```

**Important:** The executable must contain device code for the selected
backend. A CUDA-only executable cannot run its CUDA kernel on the Intel
Level Zero device. To support multiple backends in one executable, build
for each required target using a toolchain configuration that supports
those targets.

## Verification Results

The following checks were performed during development:

- [x] Custom SYCL compiler built and installed.
- [x] SYCL device enumeration detected the NVIDIA CUDA backend.
- [x] NVIDIA RTX 3050 Ti was identified as compute capability 8.6.
- [x] A SYCL test application compiled for `nvptx64-nvidia-cuda`.
- [x] The CUDA device selector selected the NVIDIA GPU.
- [x] The test kernel completed and produced the expected result.
- [x] The Intel Level Zero selector successfully ran the compatible
      test executable on Intel UHD Graphics.

Representative NVIDIA test output:

```text
Device: NVIDIA GeForce RTX 3050 Ti Laptop GPU
GPU kernel passed: 30
```

These results establish a working proof of concept. They do not establish
complete compatibility with every SYCL feature or application.

## Troubleshooting

### No CUDA device is listed

Run:

```powershell
& "$LLVM\bin\sycl-ls.exe"
```

Check the CUDA adapter installation, runtime DLL search path, NVIDIA
driver, and compiler/runtime build configuration.

### The kernel has no image for the selected device

An error such as:

```text
has no image for the selected device
```

usually means the executable does not contain compatible device code
for the selected backend. Rebuild with the appropriate target options.

### A runtime DLL cannot be found

Ensure the custom toolchain runtime directories and required CUDA
directories are available through `PATH`. Use a fresh terminal after
changing system environment variables.

### Device selector parsing errors

Use a backend-qualified selector, for example:

```powershell
$env:ONEAPI_DEVICE_SELECTOR = "cuda:gpu"
```

The selector must match an available backend and device type.

## Limitations and Future Work

This project is an experimental toolchain integration.

Potential next steps:

1. Add a reproducible build script for the custom LLVM/SYCL toolchain.
2. Add automated correctness tests for vector addition, matrix
   multiplication, and parallel reduction.
3. Benchmark equivalent workloads on NVIDIA and Intel GPUs.
4. Test supported memory models, USM, events, and synchronization.
5. Document compiler and runtime version compatibility.
6. Add CI for build checks and non-GPU tests.
7. Add GPU integration tests on appropriately configured hardware.

Performance comparisons should use equivalent workloads, correct
synchronization, and repeated measurements.

## Contributing

Issues and pull requests are welcome. Include the following when reporting
a problem:

- Windows version
- LLVM/SYCL commit or build version
- CUDA Toolkit version
- NVIDIA driver version
- GPU model
- Exact compiler command
- Full error output

## License

This repository contains work based on Intel LLVM. Consult the upstream
LLVM license and the licenses of the included components before
redistributing binaries or derived components.

Upstream project:
https://github.com/intel/llvm