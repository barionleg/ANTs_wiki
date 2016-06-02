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

If you want to use ANTs scripts, copy them from the source directory to the bin directory.


## Control multi-threading at run time

ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS


## (Optional) relocate binaries and record version information. 