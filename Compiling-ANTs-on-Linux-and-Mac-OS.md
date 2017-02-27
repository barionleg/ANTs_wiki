This installation guide is for Linux and Mac users. 

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

```
mkdir ~/bin/ants

cd ~/bin/ants

ccmake ~/code/ANTs
```

Hit 'c' to do an initial configuration. CMake will do some checking and then present options for review. Mac users will need to set `CMAKE_OSX_ARCHITECTURES` to "x86_64". 

If you are behind a firewall that blocks the git protocol, set `SuperBuild_ANTS_USE_GIT_PROTOCOL` to "OFF".

Hit 'c' again to do another round of configuration. If there are no errors, you're ready to generate the make files by pressing 'g'.

Now you are back at the command line, it's time to compile.

```
make
```

This compiles in the most resource-efficient manner. To save time, you can use multiple threads, for example:

```
make -j 2
```

will use two cores. Note that multiple threads will require more RAM as well as CPU time. If your build seems slow for the number of threads, exits with errors, or hangs up entirely, try building with a single thread.

The system will build ITK and then ANTs. Using these default settings, installation will take approximately 40 minutes. You can speed it up by turning off `RUN_LONG_TESTS`, or by turning off testing entirely.


## Copy scripts 

If you want to use ANTs scripts, copy them from the source directory `Scripts/` to the bin directory where `antsRegistration` etc are located.


## Control multi-threading at run time

Not all ANTs programs multi-thread, but many do. There is some cost to threading so running N threads won't make the programs run N times faster and the performance benefit diminishes with larger numbers of threads. By default, the number of available threads is set to the number of virtual cores, which may degrade system performance for relatively little benefit. You probably want to set the environment variable

```
ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS
```

to be at most the number of physical cores. So on an iMac with a quad-core CPU (8 virtual cores), set this variable to 4, or less than 4 if you want to save more CPU time for other processes - 2 delivers a substantial speed increase without degrading desktop performance.


## Set `PATH` and `ANTSPATH`

Assuming you've built in `~/bin/ants`, there will now be a binary directory `~/bin/ants/bin`, containing the programs (and scripts if you've included them). The scripts additionally require `ANTSPATH` to point to the bin directory **including a trailing slash**.

For the bash shell (default on Mac and some Linux), you need to set

```
export ANTSPATH=${HOME}/bin/ants/bin/
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


## Troubleshooting

It's rare for the ANTs code to have errors preventing compilation, because there are automated tests that check for successful compilation. It's very likely that any compilation problems are specific to a particular system. Most errors reported as issues relate to firewalls blocking the download of ITK, or resource issues limiting the available memory or CPU time, causing compilation to slow down or fail.


### Compilation starts but hangs with no error message

*  If the build hangs while attempting to download code, it may be because the Git protocol is blocked by a firewall. Run `ccmake` again and set `SuperBuild_ANTS_USE_GIT_PROTOCOL` to "OFF". 

* If the build hangs during compilation of some code, it's probably because the build is running out of RAM. You can reduce memory burden by compiling with fewer threads. Disabling testing may also help, set `BUILD_TESTING` to `OFF` in CMake. Alternatively, you can increase the memory available to the build process. 


### Compilation exits with error messages

* Try building with a single thread. Resource limits or timeouts can lead to incomplete compilation, resulting in errors.


### Asking for help

If you have built with a single thread using `make`, and there are still errors that you can't resolve, try searching the ANTs issues here on Github and also the [discussion forum](https://sourceforge.net/p/advants/discussion/) hosted at Sourceforge.

You may open an issue to report the error and seek help from the ANTs community. It's important to include as much relevant information as possible, including the exact version of the ANTs source (identified by the git hash), CMake, the compiler being used, and the operating system. The full output from `make` should also be included as a text file. 
