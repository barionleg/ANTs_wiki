These instructions require the Linux subsystem, which requires the Windows 10 anniversary update (Aug 2016). You may need to disable some antivirus software for some parts of the install.

## Enable the Linux subsystem on Windows 10

Follow the instructions here: https://msdn.microsoft.com/en-us/commandline/wsl/install_guide

## Install git

```
sudo apt-get -y install git
```
## Install CMake

```
sudo apt-get -y install cmake
sudo apt-get install -y cmake-curses-gui
```

## Install ANTs

From here, it should be possible to obtain the ANTs source code and compile as one would on [Linux or Mac OS](https://github.com/stnava/ANTs/wiki/Compiling-ANTs-on-Linux-and-Mac-OS#get-the-latest-code)