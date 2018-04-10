These instructions require the Linux subsystem, which requires the Windows 10 anniversary update (Aug 2016). You may need to disable some antivirus software for some parts of the install.

## Enable the Linux subsystem on Windows 10

Follow the instructions here: https://msdn.microsoft.com/en-us/commandline/wsl/install_guide

You can install your choice of Linux distributions. The instructions below are for Ubuntu, using the apt-get system to install software.

## Verify development environment

Required build tools include `gcc`, `g++`, and `make`. If these are not installed already, install them before proceeding.

```
sudo apt-get -y install build-essential
```

## Install git

```
sudo apt-get -y install git
```

## Install CMake

```
sudo apt-get -y install cmake
sudo apt-get -y install cmake-curses-gui
```

## Install Zlib

```
sudo apt-get -y install zlib1g-dev
```

## Install ANTs

From here, it should be possible to obtain the ANTs source code and compile as one would on [Linux or Mac OS](https://github.com/stnava/ANTs/wiki/Compiling-ANTs-on-Linux-and-Mac-OS#get-the-latest-code).