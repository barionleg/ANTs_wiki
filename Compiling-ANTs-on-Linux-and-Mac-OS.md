This installation guide is for Linux and Mac users. Windows users will need to install the Linux subsystem and then install Git and CMake, as detailed [here](https://github.com/stnava/ANTs/wiki/Compiling-ANTs-on-Windows-10), before proceeding to clone ANTs and compile using the instructions on this page.

Update 2019-10-01: Build instructions have been updated to reflect recent improvements to the ANTs CMake scripts.


## Compiler requirements

On all platforms, ANTs requires GNU CC 5.0 or later, or clang 3.6 or later. A current list of compilers known to build ANTs successfully can be viewed on Travis at
 
  https://travis-ci.org/ANTsX/ANTs


## Get CMake

CMake is available as source or a binary package from 

  https://cmake.org

You can also install it through a package manager for your system such as yum, apt, or Homebrew.


## Install developer tools 

### Mac OS

The exact procedure varies by Mac OS X version. For 10.11 (El Capitan), you need to first install XCode, then get the command line tools. Once XCode is installed, you can get the command line tools from the Terminal, with

```xcode-select --install```


### Linux

The developer tools are usually installed in Linux. If not, there are several routes to install the required tools, depending on the Linux installation. Look for "developer tools" packages for your Linux distribution.


## Get the latest code

```
mkdir ~/code 
cd ~/code
git clone https://github.com/stnava/ANTs.git
```

## Run CMake to configure the build

The build directory must be outside the source tree. Make a build directory, cd to it, then run `ccmake`. The build directory is not the final install location. You will set the prefix in CMake where you want to install the executables and libraries.

```
mkdir -p ~/bin/antsBuild
cd ~/bin/antsBuild
ccmake ~/code/ANTs
```

Hit 'c' to do an initial configuration. CMake will do some checking and then present options for review. You should set `CMAKE_INSTALL_PREFIX` to where you want to install ANTs. This needs to be somewhere that you have write access.

If you are behind a firewall that blocks the git protocol, set `SuperBuild_ANTS_USE_GIT_PROTOCOL` to "OFF". You may also need to replace the git protocol for the ITK build. You can do this on the command line with 
```
git config --global url."https://".insteadOf git://
```
This will tell git to use https instead of git for all of your projects.

On OS X 10.11 using clang, `CMAKE_OSX_ARCHITECTURES` needs to be left blank. Previously, this would be set to "x86_64". If CMake doesn't demand an architecture option, you should probably leave this blank.

Hit 'c' again to do another round of configuration. If there are no errors, you're ready to generate the make files by pressing 'g'.

Now you are back at the command line, it's time to compile.

```
make
```

This compiles in the most resource-efficient manner. To save time, you can use multiple threads, for example:

```
make -j 2
```

will use two cores. Note that multiple threads will require more RAM as well as CPU time. If your build seems slow for the number of threads, exits with errors, or hangs up entirely, try building with a single thread. You can also save time by turning off `RUN_LONG_TESTS` in CMake, or by turning off testing entirely.

The system will build ITK and then ANTs. 


## Install step 

After compilation completes, you will see a subdirectory `ANTS-build`. This is the location from which you will run the install.

```
cd ANTS-build
make install
```

This will copy the binaries and libraries to `bin/` and `lib/` under `CMAKE_INSTALL_PREFIX`. 


## Set `PATH` and `ANTSPATH`

Assuming your install prefix was `/opt/ants`, there will now be a binary directory `/opt/ants/bin`, containing the ANTs executables and scripts. The scripts additionally require `ANTSPATH` to point to the bin directory **including a trailing slash**.

For the bash shell (default on Mac and some Linux), you need to set

```
export ANTSPATH=/opt/ants/bin/
export PATH=${ANTSPATH}:$PATH
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


## Control multi-threading at run time

Many ANTs programs use multi-threading. By default, one thread will be generated for every CPU core on the system. This might be acceptable on a single-user machine but in a cluster environment, you will need to restrict the number of threads to be no more than the number of cores you have reserved for use.

Set the environment variable

```
ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS
```

to control the number of threads ANTs will use. 

On a desktop computer, a sensible value for this is the number of physical cores. For example, on an iMac with a quad-core CPU (8 virtual cores), set this variable to 4, or less than 4 if you want to save more CPU time for other processes.


## Troubleshooting

If you are building the latest source code, you can check the [ANTs Travis page](https://travis-ci.org/ANTsX/ANTs) to see if the code can compile successfully. 

Other common build problems:

### Compilation starts but hangs with no error message

*  If the build hangs while attempting to download code, it may be because the Git protocol is blocked by a firewall. Run `ccmake` again and set `SuperBuild_ANTS_USE_GIT_PROTOCOL` to "OFF". If that does not work, try altering your settings with `git config` to use https instead of git.

* If the build hangs during compilation of some code, it may be because the build is running out of RAM. You can reduce memory burden by compiling with fewer threads. Disabling testing may also help, set `BUILD_TESTING` to `OFF` in CMake. Alternatively, you can increase the memory available to the build process. 


### CMake complains about the compiler

If you have multiple compilers, or CMake can't find the right one for some reason, set variables before building:

```
export CC=/usr/bin/cc
export CXX=/usr/bin/c++
ccmake ~/code/ANTs
```

### Compilation runs for some time but exits with error messages

* Try building with a single thread. Resource limits or timeouts can lead to incomplete compilation, resulting in errors.

* Ensure that you have a compiler that can build ANTs. The [ANTs Travis page](https://travis-ci.org/ANTsX/ANTs) has a list of compilers that can build the latest code. 


### The build completed but I forgot to set the install prefix and can't write to the default

If you see something like

```
CMake Error at cmake_install.cmake:37 (file):
  file cannot create directory: /opt/ANTs/bin.  Maybe need administrative
  privileges.


make: *** [install] Error 1
```
and you need to install elsewhere, return to the top-level build directory (where you ran make). Run `ccmake` again and set the install prefix, then press "c" to configure and "g" to generate new make files. Then run 

```
make
cd ANTS-build
make install
```

It will take a few minutes, but it is much faster than starting over.


### Asking for help

If you have built with a single thread using `make`, and there are still errors that you can't resolve, try searching the ANTs issues here on Github and also the [discussion forum](https://sourceforge.net/p/advants/discussion/) hosted at Sourceforge.

You may open an issue to report the error and seek help from the ANTs community. It's important to include as much relevant information as possible, including the exact version of the ANTs source (identified by the git hash), CMake, the compiler being used, and the operating system. The full output from `make` should also be included as a text file. 