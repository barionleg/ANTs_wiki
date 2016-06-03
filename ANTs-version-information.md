It's good practice to record the version of ANTs you use so that you and others can reproduce your processing. It's also helpful if you need support.

The ANTs version numbers change infrequently, so it's best to record the git hash, which uniquely identifies a snapshot in the repository that others can access.

## Version information from a local git repository

If you have a local repository, you can get the full git hash from the source directory:

```
git log | head - n 1 
```

Of course, this will be incorrect if you've checked out a version other than the one you built. So it's good to do record this immediately before compiling.

Alternatively, if you have the full build directory, you can look in the file

```
ANTS-build/ProjectSourceVersionVars.cmake
```

for version information. This gives the first few characters of the git hash, enough to uniquely identify the ANTs version at the time it was checked out (and probably for quite some time after).


##  Version information from a source zip or tarball

If you download a source snapshot from the [ANTs web page](http://stnava.github.io/ANTs/], it will contain a partial git hash in the file name, eg you will have something like:

```
stnava-ANTs-v2.1.0-630-ge4b812c
```

This identifies the code as being from user `stnava`, repository `ANTs`, last version `v2.1.0`, `630` commits post that version, with git hash (`g`) `e4b812c`. When you unpack the archive it will expand to `stnava-ANTs-e4b812c`. 

If you need a snapshot it's best to use the buttons on the ANTs web page - ZIP files obtained directly from the ANTs Github page don't have version information, they are just called `ANTs-master.zip` or similar. 