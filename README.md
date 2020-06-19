# VNect Lib installation guide for Windows
[VNect](http://gvv.mpi-inf.mpg.de/projects/VNect/) is a promising pose estimation library written and developed by Dushyant Mehta and many other colleagues on the Max Planck Institute, Saarland University and Universidad Rey Juan Carlos. It generates a 3D pose estimation of a person from a given 2D image in real time either live or offline.

Through their [website](http://gvv.mpi-inf.mpg.de/projects/VNect/), you can request access to their library. Note, that they're working on a revised system, called [XNect](http://gvv.mpi-inf.mpg.de/projects/XNect/) which seems to be super robust. This guide is concerning only VNect (for now).

---
**NOTE**
Note, that this library is only working with NVIDIA CUDA, not with OpenCL or Metal due to the dependency on Caffe v1 (which supports only CUDA for now).
---

## Building dependencies

VNect uses Caffe for Windows to build it up. This contains by itself again multiple dependencies. We will go through each one of these, including:

### 1. Install Visual Studio 14 (2015)
1. Go to the [Visul Studio version archive](https://visualstudio.microsoft.com/vs/older-downloads/) and download Visual Studio 14 (2015)
2. Install it

### 2. Installing CUDA 8.0
1. Go to [NVIDIA page](https://developer.nvidia.com/cuda-80-ga2-download-archive) and download CUDA 8.0
2. Install it

### 3. Install cuDNN
For this you will need an NVIDIA developer account which you can easily create during the process.
1. Go to [NVIDIA developer archive for cuDNN](https://developer.nvidia.com/rdp/cudnn-archive) and download cuDNN v7.1.4 (May 16, 2018), for CUDA 8.0 and Windows 10
2. Unzip the ZIP archive
3. Copy the following files into the CUDA Toolkit directory (according to [this guide](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html#installwindows)).
    1. Copy `<installpath>\cuda\bin\cudnn*.dll` to `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\bin.`
    2. Copy `<installpath>\cuda\include\cudnn*.h` to `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\include.`
    3. Copy `<installpath>\cuda\lib\x64\cudnn*.lib` to `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\lib\x64`

### 4. Install Anaconda (for Python 2.7)
Caffe is quite antique and VNect is not built completely for the future, so we continue here digging in archives of projects.
1. Go to [Anaconda page](https://www.anaconda.com/products/individual#windows) and scroll to the very bottom. Select `Python 2.7 64-Bit Graphical Installer` for Windows. Alternatively, you can use this [shortcut](https://repo.anaconda.com/archive/Anaconda2-2019.10-Windows-x86_64.exe) which might have an outdated version linked (but it's a shortcut, isn't it?)
  - Alternatively, you can also install Anaconda 3 for later use if you think you will use this.
2. Install Anaconda

### 5. Install git (if not given)
1. Go to the official [Git installer download](https://git-scm.com/download/win) page and download Git 
2. Install git

### 6. Set your environment variables correct
We will now check if the libraries we use can find each other.
1. Open Windows Menu (the Windows flag) and enter `Environment Variables`. You should be able to select `Edit the system environment variables`. Select it
2. Click on `Environment variables`
3. Check the following `system variables`, following in the scheme `Variable name : Variable value`:
    - `CUDA_PATH` : `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0`
    - `CUDA_PATH_V8_0` : `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0`
    - `CUDNN_PATH` : `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\8.0`
    - `CuDnnPath` : `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\8.0`
    - `Path`:
        - `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\bin`
        - `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\libnvvp`
        - `C:\Python27\`
        - `C:\Python27\Scripts`
    - `VS140COMNTOOLS` : `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\Tools\`
4. Confirm all changes and if you have the Powershell open, restart it.

### 7. Install Chocolatey library manager
We will follow the guide from [Chocolatey](https://chocolatey.org/install)
1. Open Powershell as admin
2. Install Chocolatey by entering the following command:
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```
3. Check your installation by typing the following command into the Powershell:
```
choco
```

### 8. Install cmake
Entering the following command in your Powershell window:
```
choco install -y cmake
```

### 9. Install Caffe
We will install Caffe *without* the Ninja builder because Caffe is antique and won't be updated to support the changed API of newer Ninja versions. This removes luckily a couple of other dependencies for now (which is good) but leads to a very long building time of Caffe (which is bad).
1. Open Powershell with Admin privilidges (right click -> `Run as Administrator`)
2. Fetch the Caffe Git repository
```bash
git clone https://github.com/BVLC/caffe.git
```
3. Go into the repository and checkout the Windows branch
```
cd caffe
git checkout windows
```
**Don't close the Powershell. We will need it later!**
4. Open the script `scripts/build_win.cmd` with an editor of your choice.
5. Replace everything with the following code:
```
@echo off
@setlocal EnableDelayedExpansion

:: Default values
if DEFINED APPVEYOR (
    echo Setting Appveyor defaults
    if NOT DEFINED MSVC_VERSION set MSVC_VERSION=14
    if NOT DEFINED WITH_NINJA set WITH_NINJA=0
    if NOT DEFINED CPU_ONLY set CPU_ONLY=0
    if NOT DEFINED CUDA_ARCH_NAME set CUDA_ARCH_NAME=Auto
    if NOT DEFINED CMAKE_CONFIG set CMAKE_CONFIG=Release
    if NOT DEFINED USE_NCCL set USE_NCCL=0
    if NOT DEFINED CMAKE_BUILD_SHARED_LIBS set CMAKE_BUILD_SHARED_LIBS=0
    if NOT DEFINED PYTHON_VERSION set PYTHON_VERSION=2
    if NOT DEFINED BUILD_PYTHON set BUILD_PYTHON=1
    if NOT DEFINED BUILD_PYTHON_LAYER set BUILD_PYTHON_LAYER=1
    if NOT DEFINED BUILD_MATLAB set BUILD_MATLAB=0
    if NOT DEFINED PYTHON_EXE set PYTHON_EXE=python
    if NOT DEFINED RUN_TESTS set RUN_TESTS=1
    if NOT DEFINED RUN_LINT set RUN_LINT=1
    if NOT DEFINED RUN_INSTALL set RUN_INSTALL=1

    :: Set python 2.7 with conda as the default python
    if !PYTHON_VERSION! EQU 2 (
        set CONDA_ROOT=C:\Miniconda-x64
    )
    :: Set python 3.5 with conda as the default python
    if !PYTHON_VERSION! EQU 3 (
        set CONDA_ROOT=C:\Miniconda35-x64
    )
    set PATH=!CONDA_ROOT!;!CONDA_ROOT!\Scripts;!CONDA_ROOT!\Library\bin;!PATH!

    :: Check that we have the right python version
    !PYTHON_EXE! --version
    :: Add the required channels
    conda config --add channels conda-forge
    conda config --add channels willyd
    :: Update conda
    conda update conda -y
    :: Download other required packages
    conda install --yes cmake ninja numpy scipy protobuf==3.1.0 six scikit-image pyyaml pydotplus graphviz

    if ERRORLEVEL 1  (
      echo ERROR: Conda update or install failed
      exit /b 1
    )

    :: Install cuda and disable tests if needed
    if !WITH_CUDA! == 1 (
        call %~dp0\appveyor\appveyor_install_cuda.cmd
        set CPU_ONLY=0
        set RUN_TESTS=0
        set USE_NCCL=1
    ) else (
        set CPU_ONLY=1
    )

    :: Disable the tests in debug config
    if "%CMAKE_CONFIG%" == "Debug" (
        echo Disabling tests on appveyor with config == %CMAKE_CONFIG%
        set RUN_TESTS=0
    )

    :: Disable linting with python 3 until we find why the script fails
    if !PYTHON_VERSION! EQU 3 (
        set RUN_LINT=0
    )

) else (
    :: Change the settings here to match your setup
    :: Change MSVC_VERSION to 12 to use VS 2013
    if NOT DEFINED MSVC_VERSION set MSVC_VERSION=14
    :: Change to 1 to use Ninja generator (builds much faster)
    if NOT DEFINED WITH_NINJA set WITH_NINJA=0
    :: Change to 1 to build caffe without CUDA support
    if NOT DEFINED CPU_ONLY set CPU_ONLY=0
    :: Change to generate CUDA code for one of the following GPU architectures
    :: [Fermi  Kepler  Maxwell  Pascal  All]
    if NOT DEFINED CUDA_ARCH_NAME set CUDA_ARCH_NAME=Auto
    :: Change to Debug to build Debug. This is only relevant for the Ninja generator the Visual Studio generator will generate both Debug and Release configs
    if NOT DEFINED CMAKE_CONFIG set CMAKE_CONFIG=Release
    :: Set to 1 to use NCCL
    if NOT DEFINED USE_NCCL set USE_NCCL=1
    :: Change to 1 to build a caffe.dll
    if NOT DEFINED CMAKE_BUILD_SHARED_LIBS set CMAKE_BUILD_SHARED_LIBS=0
    :: Change to 3 if using python 3.5 (only 2.7 and 3.5 are supported)
    if NOT DEFINED PYTHON_VERSION set PYTHON_VERSION=2
    :: Change these options for your needs.
    if NOT DEFINED BUILD_PYTHON set BUILD_PYTHON=1
    if NOT DEFINED BUILD_PYTHON_LAYER set BUILD_PYTHON_LAYER=1
    if NOT DEFINED BUILD_MATLAB set BUILD_MATLAB=0
    :: If python is on your path leave this alone
    if NOT DEFINED PYTHON_EXE set PYTHON_EXE=python
    :: Run the tests
    if NOT DEFINED RUN_TESTS set RUN_TESTS=0
    :: Run lint
    if NOT DEFINED RUN_LINT set RUN_LINT=0
    :: Build the install target
    if NOT DEFINED RUN_INSTALL set RUN_INSTALL=1
)

:: Set the appropriate CMake generator
:: Use the exclamation mark ! below to delay the
:: expansion of CMAKE_GENERATOR
if %WITH_NINJA% EQU 0 (
    if "%MSVC_VERSION%"=="14" (
        set CMAKE_GENERATOR=Visual Studio 14 2015 Win64
    )
    if "%MSVC_VERSION%"=="12" (
        set CMAKE_GENERATOR=Visual Studio 12 2013 Win64
    )
    if "!CMAKE_GENERATOR!"=="" (
        echo ERROR: Unsupported MSVC version
        exit /B 1
    )
) else (
    set CMAKE_GENERATOR=Ninja
)

echo INFO: ============================================================
echo INFO: Summary:
echo INFO: ============================================================
echo INFO: MSVC_VERSION               = !MSVC_VERSION!
echo INFO: WITH_NINJA                 = !WITH_NINJA!
echo INFO: CMAKE_GENERATOR            = "!CMAKE_GENERATOR!"
echo INFO: CPU_ONLY                   = !CPU_ONLY!
echo INFO: CUDA_ARCH_NAME             = !CUDA_ARCH_NAME!
echo INFO: CMAKE_CONFIG               = !CMAKE_CONFIG!
echo INFO: USE_NCCL                   = !USE_NCCL!
echo INFO: CMAKE_BUILD_SHARED_LIBS    = !CMAKE_BUILD_SHARED_LIBS!
echo INFO: PYTHON_VERSION             = !PYTHON_VERSION!
echo INFO: BUILD_PYTHON               = !BUILD_PYTHON!
echo INFO: BUILD_PYTHON_LAYER         = !BUILD_PYTHON_LAYER!
echo INFO: BUILD_MATLAB               = !BUILD_MATLAB!
echo INFO: PYTHON_EXE                 = "!PYTHON_EXE!"
echo INFO: RUN_TESTS                  = !RUN_TESTS!
echo INFO: RUN_LINT                   = !RUN_LINT!
echo INFO: RUN_INSTALL                = !RUN_INSTALL!
echo INFO: ============================================================

:: Build and exectute the tests
:: Do not run the tests with shared library
if !RUN_TESTS! EQU 1 (
    if %CMAKE_BUILD_SHARED_LIBS% EQU 1 (
        echo WARNING: Disabling tests with shared library build
        set RUN_TESTS=0
    )
)

if NOT EXIST build mkdir build
pushd build

:: Setup the environement for VS x64
set batch_file=!VS%MSVC_VERSION%0COMNTOOLS!..\..\VC\vcvarsall.bat
call "%batch_file%" amd64

:: Configure using cmake and using the caffe-builder dependencies
:: Add -DCUDNN_ROOT=C:/Projects/caffe/cudnn-8.0-windows10-x64-v5.1/cuda ^
:: below to use cuDNN
cmake -G"!CMAKE_GENERATOR!" ^
      -DBLAS=Open ^
      -DCMAKE_BUILD_TYPE:STRING=%CMAKE_CONFIG% ^
      -DBUILD_SHARED_LIBS:BOOL=%CMAKE_BUILD_SHARED_LIBS% ^
      -DBUILD_python:BOOL=%BUILD_PYTHON% ^
      -DBUILD_python_layer:BOOL=%BUILD_PYTHON_LAYER% ^
      -DBUILD_matlab:BOOL=%BUILD_MATLAB% ^
      -DCPU_ONLY:BOOL=%CPU_ONLY% ^
      -DCOPY_PREREQUISITES:BOOL=1 ^
      -DINSTALL_PREREQUISITES:BOOL=1 ^
      -DUSE_NCCL:BOOL=!USE_NCCL! ^
      -DCUDA_ARCH_NAME:STRING=%CUDA_ARCH_NAME% ^
      "%~dp0\.."

if ERRORLEVEL 1 (
  echo ERROR: Configure failed
  exit /b 1
)

:: Lint
if %RUN_LINT% EQU 1 (
    cmake --build . --target lint  --config %CMAKE_CONFIG%
)

if ERRORLEVEL 1 (
  echo ERROR: Lint failed
  exit /b 1
)

:: Build the library and tools
cmake --build . --config %CMAKE_CONFIG%

if ERRORLEVEL 1 (
  echo ERROR: Build failed
  exit /b 1
)

:: Build and exectute the tests
if !RUN_TESTS! EQU 1 (
    cmake --build . --target runtest --config %CMAKE_CONFIG%

    if ERRORLEVEL 1 (
        echo ERROR: Tests failed
        exit /b 1
    )

    if %BUILD_PYTHON% EQU 1 (
        if %BUILD_PYTHON_LAYER% EQU 1 (
            :: Run python tests only in Release build since
            :: the _caffe module is _caffe-d is debug
            if "%CMAKE_CONFIG%"=="Release" (
                :: Run the python tests
                cmake --build . --target pytest

                if ERRORLEVEL 1 (
                    echo ERROR: Python tests failed
                    exit /b 1
                )
            )
        )
    )
)

if %RUN_INSTALL% EQU 1 (
    cmake --build . --target install --config %CMAKE_CONFIG%
)

popd
@endlocal
```
Here's an explanation of the changes:
- `if NOT DEFINED MSVC_VERSION set MSVC_VERSION=14`: We set the Visual Studio Code version to `Visual Studio 14`.
- `if NOT DEFINED WITH_NINJA set WITH_NINJA=0`: We don't use the Ninja builder.
- `if NOT DEFINED CPU_ONLY set CPU_ONLY=0`: We want to build for the GPU. So disable the CPU only usage.
- `if NOT DEFINED USE_NCCL set USE_NCCL=1`: Enable building with cuDNN support.
- `if NOT DEFINED PYTHON_VERSION set PYTHON_VERSION=2`: We use old Python version 2.
- `if NOT DEFINED BUILD_PYTHON set BUILD_PYTHON=1`: We build the Python library as well.
- `if NOT DEFINED BUILD_PYTHON_LAYER set BUILD_PYTHON_LAYER=1`: Yeah... let's build the Python Caffe layer as well.
- `if NOT DEFINED BUILD_MATLAB set BUILD_MATLAB=0`: We don't have Matlab installed.
- `if NOT DEFINED RUN_TESTS set RUN_TESTS=0`: Disable running tests due to cuDNN support (somehow the makers have disabled the tests then).
- `if NOT DEFINED RUN_INSTALL set RUN_INSTALL=1`: Install it as global library.
All these changes are followed right in the block after `) else (`.

6. Go back to your Powershell window. Install additional dependencies for running it with Anaconda. Paste each line separately into the Powershell.
```
conda config --add channels conda-forge
conda config --add channels willyd
conda install --yes cmake ninja numpy scipy protobuf==3.1.0 six scikit-image pyyaml pydotplus graphviz
```
7. Build Caffe by entering the following command in the Powershell. (I assume you're still in the `caffe/` folder of the Git repository. Double check for it!)
```
./scripts\build_win.cmd
```
### 10. Install VNect library
This is quite straightforward, so you just follow the README.md
1. Download VNect library and place it where you can find it. I recommend for now to place it into `<your-home-folder>/Projects/vnect-lib`
2. After downloading, open your Powershell as Admin again, navigate to the project folder. When you follow my recommendation, it would be:
```
cd ~/Projects/vnect-lib
```
3. Create cmake folder and go into that
```
mkdir build
cd build
```
4. Run the cmake command
```
cmake -G "Visual Studio 14 Win64" -C C:\Users\<your-username>\.caffe\dependencies\libraries_v140_x64_py27_1.1.0\libraries\caffe-builder-config.cmake ..
```
