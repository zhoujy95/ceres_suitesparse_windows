[![Build Status](https://img.shields.io/travis/jlblancoc/suitesparse-metis-for-windows/master.svg?label=Travis)](https://travis-ci.org/jlblancoc/suitesparse-metis-for-windows/builds)
[![Grunt status](https://img.shields.io/appveyor/ci/jlblancoc/suitesparse-metis-for-windows/master.svg?label=Appveyor)](https://ci.appveyor.com/project/jlblancoc/suitesparse-metis-for-windows/history)


CMake scripts for painless usage of Tim Davis' [SuiteSparse](http://faculty.cse.tamu.edu/davis/suitesparse.html) (CHOLMOD,UMFPACK,AMD,LDL,SPQR,...) and [METIS](http://glaros.dtc.umn.edu/gkhome/views/metis) from Visual Studio and the rest of Windows/Linux/OSX IDEs supported by CMake. The project includes precompiled BLAS/LAPACK DLLs for easy use with Visual C++. Licensed under BSD 3-Clause License.

The goal is using one single CMake code to build against *SuiteSparse* in standard Linux package systems (e.g. `libsuitesparse-dev`) and in manual compilations under Windows.

**Credits:** Jose Luis Blanco (Universidad de Almeria); Jerome Esnault (INRIA); [@NeroBurner](https://github.com/NeroBurner)

![logo](https://raw.githubusercontent.com/jlblancoc/suitesparse-metis-for-windows/master/docs/logo.png)

## 1. Instructions

  * (1) Install [CMake](https://www.cmake.org/).
  * (2) Only for Linux/Mac: Install LAPACK & BLAS. In Debian/Ubuntu: `sudo apt-get install liblapack-dev libblas-dev`
  * (3) Clone or download this project ([latest release](https://github.com/jlblancoc/suitesparse-metis-for-windows/releases)) and extract it somewhere in your disk, say `SP_ROOT`.
	  * (OPTIONAL) CMake will download SuiteSparse sources automatically for you (skip to step 4), but you may do it manually if preferred:
        * Populate the directories within `SP_ROOT` with the original sources from each project:
          * *`SuiteSparse`:*
            * Download [SuiteSparse-X.Y.Z.tar.gz](http://faculty.cse.tamu.edu/davis/suitesparse.html) from Tim Davis' webpage.
            * Extract it.
            * Merge (e.g. copy and paste from Windows Explorer) the tree `SuiteSparse/*` into `SP_ROOT/SuiteSparse/*`.
            * Make sure of **looking for patches** in [the original webpage](http://faculty.cse.tamu.edu/davis/suitesparse.html) and apply them to prevent build errors.
          * *`METIS`:*  (Optional, only if need METIS for partitioning)
            * Download [metis-X.Y.Z.tar.gz](http://glaros.dtc.umn.edu/gkhome/metis/metis/download).
            * Extract it.
            * Merge the tree `metis-X.Y.Z/*` into `SP_ROOT/metis/*`.
            * Add the command `cmake_policy(SET CMP0022 NEW)` right after the line `project(METIS)` in `metis/CMakeLists.txt`.

  * (4) **Run CMake** (cmake-gui), then:
      * Set the "Source code" directory to `SP_ROOT`
	  * Set the "Build" directory to any empty directory, typically `SP_ROOT/build`
	  * Press "Configure", change anything (if needed)
      * **Important**: I recommend setting the `CMAKE_INSTALL_PREFIX` to some other directory different than "Program Files" or "/usr/local" so the INSTALL command does not require Admin privileges. By default it will point to `SP_ROOT/build/install`.
      * If you have an error like: "Cannot find source file: GKlib/conf/check_thread_storage.c", then manually adjust `GKLIB_PATH` to the correct path `SP_ROOT/metis/GKlib`.
      * `HAVE_COMPLEX` is OFF by default to avoid errors related to complex numbers in some compilers.
	  * Press "Generate".
  * (5) **Compile and install:**
    * In Visual Studio, open `SuiteSparseProject.sln` and build the `INSTALL` project in Debug and Release. You may get hundreds of warnings, but it's ok.
    * In Unix: Just execute `make install` or `sudo make install` if you did set the install prefix to `/usr/*`

  * (6) Notice that a file `SuiteSparseConfig.cmake` should be located in your install directory. It will be required for your programs to correctly build and link against `SuiteSparse`.

  * (7) Only for Windows: You will have to append `CMAKE_INSTALL_PREFIX\lib*\lapack_blas_windows\` and `CMAKE_INSTALL_PREFIX\lib*` to the environment variable `PATH` before executing any program, for Windows to localize the required BLAS/Fortran libraries (`.DLL`s).


## 2. Test program

Example CMake programs are provided for testing, based on Tim Davis' code in his manual:
  * [example-projects](https://github.com/jlblancoc/suitesparse-metis-for-windows/tree/master/example-projects)

An example to test CUDA support can be found [here](https://gist.github.com/andr3wmac/78d294844484cb48342f88ef03e2776a).


## 3. Integration in your code (unique code for Windows/Linux)


  * Add a block like this to your CMake code (see complete [example](https://github.com/jlblancoc/suitesparse-metis-for-windows/blob/master/example-projects/cholmod/CMakeLists.txt)), and set the CMake variable `SuiteSparse_DIR` to
  `SP_INSTALL_DIR/lib/cmake/suitesparse-5.4.0/` or the equivalent place where `SuiteSparse-config.cmake` was installed.

    ```
    # ------------------------------------------------------------------
    # Detect SuiteSparse libraries:
    # If not found automatically, set SuiteSparse_DIR in CMake to the
    # directory where SuiteSparse-config.cmake was installed.
    # ------------------------------------------------------------------
    find_package(SuiteSparse CONFIG REQUIRED)
    target_link_libraries(MY_PROGRAM PRIVATE ${SuiteSparse_LIBRARIES})

    # or directly add only the libs you need:
    target_link_libraries(MY_PROGRAM PRIVATE
      	SuiteSparse::suitesparseconfig
      	SuiteSparse::amd
      	SuiteSparse::btf
      	SuiteSparse::camd
      	SuiteSparse::ccolamd
      	SuiteSparse::colamd
      	SuiteSparse::cholmod
      	SuiteSparse::cxsparse
      	SuiteSparse::klu
      	SuiteSparse::ldl
      	SuiteSparse::umfpack
      	SuiteSparse::spqr
      	SuiteSparse::metis
      )
    ```

  * In Windows, if you have a CMake error like:

    ```
    Could not find a package configuration file provided by "LAPACK" with any
     of the following names:

       LAPACKConfig.cmake
       lapack-config.cmake
    ```

    set the CMake variable `LAPACK_DIR` to `SP_ROOT/lapack_windows/x64/` (or `x32` for 32bit builds).


## 4. Why did you create this project?

Porting `SuiteSparse` to CMake wasn't trivial because this package makes extensive usage of a _one-source-multiple-objects_ philosophy by compiling the same sources with different compiler flags, and this ain't directly possible to CMake's design.

My workaround to this limitation includes auxiliary Python scripts and dummy source files, so the whole thing became large enough to be worth publishing online so many others may benefit.
