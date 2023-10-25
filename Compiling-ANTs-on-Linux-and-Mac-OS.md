This installation guide is for Linux and Mac users. Windows users will need to install the Linux subsystem and then install Git and CMake, as detailed [here](https://github.com/stnava/ANTs/wiki/Compiling-ANTs-on-Windows-10), before proceeding to clone ANTs and compile using the instructions on this page.

The instructions here will perform a "SuperBuild", which will automatically build the correct versions of ITK (required) and VTK (optional) for ANTs. Most ANTs users will find it easiest to use the SuperBuild. Advanced users can use [system ITK or VTK](https://github.com/ANTsX/ANTs/wiki/Compiling-ANTs-on-Linux-and-Mac-OS#using-system-itk-or-vtk).

# SuperBuild quick reference

A downloadable script to build and install ANTs locally is [available here](https://github.com/cookpa/antsInstallExample).

Here is a very minimal example. This example works if you have all the necessary tools, including CMake and a supported compiler. The system requirements and installation steps are discussed in more detail below.

```
workingDir=${PWD}
git clone https://github.com/ANTsX/ANTs.git
mkdir build install
cd build
cmake \
    -DCMAKE_INSTALL_PREFIX=${workingDir}/install \
    ../ANTs 2>&1 | tee cmake.log
make -j 4 2>&1 | tee build.log
cd ANTS-build
make install 2>&1 | tee install.log
```

# Compiler requirements

ANTs requires a compiler that is able to [compile ITK](https://github.com/InsightSoftwareConsortium/ITK/blob/master/Documentation/SupportedCompilers.md). The compilers used by the developers are usually GCC and Apple clang. If compilation fails on a very new or old version of these compilers, try GCC 7 or Apple clang 12.0.0, which are known to work as of March 2022.


# CMake requirements

CMake is available as source or a binary package from 

  https://cmake.org

You can also install it through a package manager for your system such as yum, apt, or Homebrew.

The [supported CMake versions](https://github.com/ANTsX/ANTs/blob/fc16efb8de42aa20955f694c31ea0af491e78d9e/CMakeLists.txt#L1-L3) usually mirror those for ITK. Configuration of ANTs will fail immediately if the CMake version is not supported.


# Installing developer tools 

## Mac OS

First install XCode, then get the command line tools from the Terminal with

```xcode-select --install```


## Linux

The developer tools are usually installed in Linux. If not, there are several routes to install the required tools, depending on the Linux installation. Look for "developer tools" packages for your Linux distribution.


# Get the latest code

```
git clone https://github.com/ANTsX/ANTs.git
```

You can also download code snapshots as a ZIP file from Github, but you will still need `git` installed for the SuperBuild to work.


# Run CMake to configure the build

The build directory must be outside the source tree. Make a build directory, cd to it, then run `cmake` with command line options or `ccmake` for the GUI. The build directory is not the final install location. You will set the prefix in CMake where you want to install the executables and libraries, but that is a separate command after the build completes.

```
# make a build and install dir
mkdir build install
cd build
ccmake ../ANTs
```

In the GUI, hit 'c' to do an initial configuration. CMake will do some checking and then present options for review. You should set `CMAKE_INSTALL_PREFIX` to where you want to install ANTs. This needs to be somewhere that you have write access. Set any other options you need here as well.

Hit 'c' again to do another round of configuration. If there are no errors, you're ready to generate the make files by pressing 'g'.

Now you are back at the command line, it's time to compile.

# Build step

```
make 2>&1 | tee build.log
```

The last part of the command here records the terminal output to a file `build.log`. This is optional, but you must do this if you need to report an issue. If you don't have the `tee` command installed, you can do

```
make > build.log 2>&1 
```

but you will not see output to the terminal. 

To speed up compilation, you can use multiple threads, for example:

```
make -j 2 2>&1 | tee build.log
```

will use two threads. Note that multiple threads will require more RAM as well as CPU resources. If your build seems slow for the number of threads, exits with errors, or hangs up entirely, try building with a single thread. You can also save time by turning off `RUN_LONG_TESTS` in CMake, or by turning off testing entirely.


# Checking compilation success

If all went well, `make` will exit with code 0, after printing

```
[100%] Built target ANTS
```

You can also check for the empty file `CMakeFiles/ANTS-complete` under the build directory.


# Install step 

After compilation completes, you will see a subdirectory `ANTS-build`. This is the location from which you will run the install.

```
cd ANTS-build
make install 2>&1 | tee install.log
```

This will copy the binaries and libraries to `bin/` and `lib/` under `CMAKE_INSTALL_PREFIX`. 


# Post installation: set environment variable `PATH`

Set the PATH to include the bin directory under whatever you used for `CMAKE_INSTALL_PREFIX`. For example, if your install prefix was `/opt/ants`, there will now be a binary directory `/opt/ants/bin`, containing the ANTs executables and scripts.

```
export PATH=/opt/ants/bin:$PATH
```

Now check this worked correctly:

```
which antsRegistration
```

should print out the full path to `antsRegistration`, and

```
antsRegistrationSyN.sh
```

should print out the usage for that script. You can put the above variable definitions in your shell initialization file, so future sessions will have them set automatically. On a Mac, this is usually `~/.profile`, on Linux `~/.bash_profile`.


## ANTSPATH (deprecated in v2.5.0)

ANTs scripts prior to v2.5.0 require `ANTSPATH` as well as PATH, where `ANTSPATH` points to the bin directory **including a trailing slash**.

```
export ANTSPATH=/opt/ants/bin/
export PATH=${ANTSPATH}:$PATH
```

# Post installation: control multi-threading at run time

Many ANTs programs use multi-threading. By default, one thread will be generated for every CPU core on the system. This might be acceptable on a single-user machine but in a cluster environment, you will need to restrict the number of threads to be no more than the number of cores you have reserved for use.

Set the environment variable

```
ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS
```

to control the number of threads ANTs will use. 

On a desktop computer, a sensible value for this is the number of physical cores. For example, on an iMac with a quad-core CPU (8 virtual cores), set this variable to 4, or less than 4 if you want to save more CPU time for other processes.


# Troubleshooting

## git error messages while cloning ITK or VTK

Github has made some changes to its [connection protocols](https://github.blog/2021-09-01-improving-git-protocol-security-github/) that might cause the build to fail. 

Older ANTs code used the deprecated unencrypted `git://` protocol by default. This can be changed by setting the option `SuperBuild_ANTS_USE_GIT_PROTOCOL` `OFF` in CMake or `ccmake`. The latest code should use `https://` consistently.


## CMake Error: install(EXPORT "ITKTargets" ...) includes target "gdcmjpeg8" more than once in the export set.

This is a known issue with CMake 3.19.0 and 3.19.1.

https://gitlab.kitware.com/cmake/cmake/-/issues/21529

Compile with CMake 3.19.2 or later.


## Compilation starts but hangs with no error message

If the build hangs during compilation of some code, it may be because the build is running out of RAM. You can reduce memory burden by compiling with fewer threads. Disabling testing may also help, set `BUILD_TESTING` to `OFF` in CMake. Alternatively, you can increase the memory available to the build process. 


## CMake complains about the compiler

If you have multiple compilers, or CMake can't find the right one for some reason, set variables before building:

```
export CC=/usr/bin/cc
export CXX=/usr/bin/c++
ccmake ~/code/ANTs
```

## Compilation runs for some time but exits with error messages

* Try building with a single thread. Resource limits or timeouts can lead to incomplete compilation, resulting in errors.

* Ensure that you have a compiler that can build ANTs. If using a brand new or very old compiler, try one that is known to build ITK.

* If using system ITK or VTK, try a SuperBuild. Through the efforts of developers and contributors, ANTs tries to keep up with the innovations and C++ implementation updates in ITK. This means the ITK version required by ANTs changes fairly frequently and is often not backwards compatible.

* If building with testing on, there can be difficulties reaching the servers to download test data. ANTs prior to 2.4.0 will not work with testing on, because it uses deprecated MD5 hashes to identify the test data. 


## Compilation fails or run time errors occur with "Illegal Instruction" errors

This is a result of the compiler using instruction sets that are not supported by some CPUs. By default, the target architecture is "corei7", which works for most modern PCs and Macs. But some users may need to change the compiler architecture options used by CMake by adding 

```
-DSuperBuild_ANTS_C_OPTIMIZATION_FLAGS="-mtune=native -march=native" \
-DSuperBuild_ANTS_CXX_OPTIMIZATION_FLAGS="-mtune=native -march=native"
```

to the `cmake` call. This will target the build to the machine used to compile ANTs. To make a more portable build, replace `-march=native` with "-march=x86-64". 

Related issue: [764](https://github.com/ANTsX/ANTs/issues/764).


## The build completed but I forgot to set the install prefix and can't write to the default

If you see something like

```
CMake Error at cmake_install.cmake:37 (file):
  file cannot create directory: /opt/ants/bin.  Maybe need administrative
  privileges.


make: *** [install] Error 1
```

The fastest way to fix this is with the `DESTDIR` variable:

```
make install DESTDIR=/new/install/dir
```

This will install the binaries under `${DESTDIR}/${CMAKE_INSTALL_PREFIX}`, which by default is `${DESTDIR}/opt/ants/bin`. Alternatively, you can reconfigure the install directory with CMake:

```
cmake -DCMAKE_INSTALL_PREFIX=/new/install/dir .
make install
```


## Asking for help

If you still have problems, try searching the ANTs issues here on Github.

You may open an issue to report the error and seek help from the ANTs community. Please see the issue template for build issues, and include all the relevant attachments.


# Advanced topics

For developers and advanced users who are not using the default Superbuild.

## Building with VTK

VTK is required for the `antsSurf` program. VTK can be downloaded and built as part of the Superbuild, by setting `USE_VTK=ON` in CMake.

## Using system ITK or VTK

If you wish to use a pre-compiled system version of either ITK or VTK, it must be a compatible version for the ANTs source you are working from. The ITK version changes most frequently. The required ITK version can be found in the ANTs source [here](https://github.com/ANTsX/ANTs/blob/master/SuperBuild/External_ITKv5.cmake) and the VTK version [here](https://github.com/ANTsX/ANTs/blob/master/SuperBuild/External_VTK.cmake). These files also show the [required modules for building ITK](https://github.com/ANTsX/ANTs/blob/d30526f9cb5159bc0a3e9011f7ae5f409b3634c8/SuperBuild/External_ITKv5.cmake#L114-L138) and VTK for compatibility with ANTs. 


