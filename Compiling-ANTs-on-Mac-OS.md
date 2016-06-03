## Get CMake

CMake is available as source or a binary package from 

  https://cmake.org


## Install developer tools 

The exact procedure varies by Mac OS X version. For 10.11 (El Capitan), you need to first install XCode, then get the command line tools. Once XCode is installed, you can get the command line tools from the Terminal, with

```xcode-select --install```


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

Hit 'c' to do an initial configuration. CMake will do some checking and then present options for review. Set `CMAKE_OSX_ARCHITECTURES` to "x86_64". 

If you are behind a firewall that blocks the git protocol, set SuperBuild_ANTS_USE_GIT_PROTOCOL to "OFF".

Hit 'c' again to do another round of configuration. If there are no errors, you're ready to generate the make files by pressing 'g'.

Now you are back at the command line, it's time to compile.

```
make -j 4
```

The option `-j` lets `make` run multiple threads, which speeds up compilation. Here I've used 4, which is suitable for a quad-core iMac. 

The system will build ITK and then ANTs. Using these default settings, installation will take approximately 1 hour.


## Copy scripts 

If you want to use ANTs scripts, copy them from the source directory `Scripts/` to the bin directory where `antsRegistration` etc are located.


## Control multi-threading at run time

Not all ANTs programs multi-thread, but many do. There is some cost to threading so running N threads won't make the programs run N times faster and the performance benefit diminishes with larger numbers of threads. By default, the number of available threads is set to the number of virtual cores, which may degrade system performance for relatively little benefit. You probably want to set the environment variable

```
ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS
```

to be at most the number of physical cores. So on an iMac with a quad-core CPU (8 virtual cores), set this variable to 4, or less than 4 if you want to save more CPU time for other processes - 2 delivers a substantial speed increase without degrading desktop performance.


## (Optional) relocate binaries and record version information. 

If you built with static libraries (the default, and usually what you want), you can copy the binaries directory

```
~/bin/ants/bin
```

elsewhere on your system, or to another Mac with the same architecture. You can then clean up the rest of the binaries in your compile directory. Before doing this, record the version of ANTs you just built. 
