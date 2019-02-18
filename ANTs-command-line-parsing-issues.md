This page lists some known issues with ANTs command line parsing. 


## Shell expansion of square brackets

The bash shell interprets arguments containing square brackets as a pattern, and attempts to perform [filename expansion](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_04.html#sect_03_04_08). This creates a problem if there is a file matching one of the characters in the brackets. For example, the argument `-c [100x100x100]` passed to `antsRegistration` will be incorrect if a file named "1", "0", or "x" exists in the current working directory. 

There's various strategies to deal with this:

1. Escape brackets with quotes `-c "[100x100x100]"` or backslashes `-c \[100x100x100\]`. 

2. Use `set -f` in a job script to disable pattern expansion. But this will also disable other pattern matching, so for example `AverageImages 3 avg.nii.gz 0 *.nii.gz` would fail unless the matching was re-enabled with `set +f`.

3. Avoid using single-character file names.



## File names containing multiple extensions

This should not be a problem any longer but may affect older versions. Multiple extensions, or even just periods, in file names used to cause errors. Some known issues have been fixed (for example [568](https://github.com/ANTsX/ANTs/pull/568) and [658](https://github.com/ANTsX/ANTs/pull/658)).