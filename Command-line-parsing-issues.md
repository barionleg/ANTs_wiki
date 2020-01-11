This page lists some known issues with ANTs command line parsing. 


## Shell expansion of square brackets

Bash and related shells interpret arguments containing square brackets as a pattern, and attempt to perform [filename expansion](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_04.html#sect_03_04_08). This creates a problem if there is a file matching one of the characters in the brackets. For example, the argument `-c [100x100x100]` passed to `antsRegistration` will be incorrect if a file named "1", "0", or "x" exists in the current working directory. 

### Solution for ANTs executables

The easiest way to avoid glob problems is to include white space inside the square brackets in ANTs parameters when calling ANTs executables. For example `-c [ 100x100x100 ]` or `-t SyN[ 0.1 ]` passed to `antsRegistration`.
If you are using `zsh` as your shell, you may prepend the call to your executable with the `noglob` command. Instead of calling `antsRegistration`, for example, call `noglob antsRegistration`.

### Solution for ANTs scripts

Script arguments with spaced brackets should be quoted, for example `antsCorticalThickness -c "WM[ 4 ]"`.

ANTs scripts have been updated to avoid unwanted filename expansion ([f154285](https://github.com/ANTsX/ANTs/commit/f154285a72c357366a4ad99bd2eef946eb843878)). 


## File names containing multiple extensions

This should not be a problem any longer but may affect older versions. Multiple extensions, or even just periods, in file names used to cause errors. Some known issues have been fixed (for example [568](https://github.com/ANTsX/ANTs/pull/568) and [658](https://github.com/ANTsX/ANTs/pull/658)).