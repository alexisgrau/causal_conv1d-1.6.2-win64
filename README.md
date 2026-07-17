# Building `causal-conv1d` on Windows (Python 3.13 / CUDA 13)

This repository provides a precompiled Windows build of
**causal-conv1d** along with the steps I used to compile it
successfully.

The main goal of this guide is to demonstrate that **it is possible to
build `causal-conv1d` without installing the full NVIDIA CUDA Toolkit**.
Instead, the required CUDA compiler (`nvcc`) and related components are
installed directly from Python packages.

> **Note:** This procedure was tested with the environment described
> below. Other versions of Python, PyTorch, CUDA, or Visual Studio may
> require small adjustments.

------------------------------------------------------------------------

## Tested Environment

### Operating System

-   Windows 11 x64
-   PowerShell
-   Visual Studio Community with C++ Build Tools installed

### Python

``` powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**Python version**

``` text
Python 3.13.14
```

### GPU

``` text
NVIDIA GeForce RTX 3060 12 GB
Driver Version: 595.97
CUDA Version: 13.2
```

### Successfully Built Version

``` text
causal-conv1d==1.6.2.post1
```

------------------------------------------------------------------------

## Prerequisites

Before attempting to compile `causal-conv1d`, I already had a fully
working CUDA-enabled PyTorch/Unsloth environment.

Relevant package versions included:

``` text
torch==2.10.0+cu130
torchvision==0.25.0+cu130
torchaudio==2.11.0+cu130
unsloth==2026.7.2
triton-windows==3.7.1.post27
transformers==5.5.0
xformers==0.0.35
```

------------------------------------------------------------------------

## Install CUDA Compiler Packages

``` powershell
pip cache purge
pip install nvidia-cuda-nvcc nvidia-cuda-cccl
```

Unlike most tutorials, **the official NVIDIA CUDA Toolkit was NOT
installed**.

------------------------------------------------------------------------

## Load the Visual Studio Build Environment

``` powershell
$vcvars = "{ProgramFiles}\Microsoft Visual Studio\<VERSION>\Community\VC\Auxiliary\Build\vcvars64.bat"

if (Test-Path $vcvars) {
    cmd /c "`"$vcvars`" && set" | ForEach-Object {
        if ($_ -match '^(?<Name>.*?)=(?<Value>.*)$') {
            Set-Item "env:$($Matches.Name)" $Matches.Value
        }
    }
}
```

------------------------------------------------------------------------

## Configure CUDA Environment Variables

Replace `<YOUR_VENV>` with the path to your virtual environment.

``` powershell
$env:NVCC_PREPEND_FLAGS="-I <YOUR_VENV>\Lib\site-packages\nvidia\cu13\include\cccl -Xcompiler /Zc:preprocessor -DCCCL_IGNORE_MSVC_TRADITIONAL_PREPROCESSOR_WARNING"

$env:CFLAGS="/Zc:preprocessor"
$env:CXXFLAGS="/Zc:preprocessor"

$env:CUDA_HOME="<YOUR_VENV>\Lib\site-packages\nvidia\cu13"
$env:PATH="<YOUR_VENV>\Lib\site-packages\nvidia\cu13\bin;$env:PATH"

$env:FORCE_CUDA="1"
$env:DISTUTILS_USE_SDK="1"
$env:PLATDIR="win-amd64"
$env:CMAKE_GENERATOR_PLATFORM="x64"
```

------------------------------------------------------------------------

## Build

``` powershell
pip install causal-conv1d --no-build-isolation
```

------------------------------------------------------------------------

## Verify the Installation

``` powershell
pip show causal-conv1d

python -c "import causal_conv1d; print('Import successful! Version:', causal_conv1d.__version__)"
```

Expected output:

``` text
Import successful! Version: 1.6.2.post1
```

------------------------------------------------------------------------

## Notes

-   No NVIDIA CUDA Toolkit installation was required.
-   Only the `nvidia-cuda-nvcc` and `nvidia-cuda-cccl` Python packages
    were used.
-   Microsoft Visual Studio C++ Build Tools are still required.
-   A working CUDA-enabled PyTorch installation is recommended.
-   `--no-build-isolation` is important so the build uses your current
    environment.

------------------------------------------------------------------------

## Tested Versions

  Component       Version
  --------------- -----------------------
  Windows         11 x64
  Python          3.13.14
  GPU             NVIDIA RTX 3060 12 GB
  NVIDIA Driver   595.97
  CUDA Driver     13.2
  PyTorch         2.10.0+cu130
  Unsloth         2026.7.2
  causal-conv1d   1.6.2.post1

Hopefully this saves someone else a few hours of trial and error when
building `causal-conv1d` on Windows.
