It's good practice to record the version of ANTs you use so that you and others can reproduce your processing. It's also helpful if you need support.

## Version information in compiled binaries

If you clone the ANTs git repository and build against that, the version information will be encoded into the binaries. You can recover this later with

```
antsRegistration --version
```


## Version information from a local git repository

If you can't build ANTs, you can get the full git hash from the source directory:

```
git log | head - n 1 
```

##  Version information from a source zip or tarball

If you download a source snapshot from the [ANTs web page](http://stnava.github.io/ANTs/], it will contain a partial git hash in the file name, eg you will have something like:

```
stnava-ANTs-v2.1.0-630-ge4b812c
```

This identifies the code as being from user `stnava`, repository `ANTs`, last version `v2.1.0`, `630` commits post that version, with git hash (`g`) `e4b812c`. When you unpack the archive it will expand to `stnava-ANTs-e4b812c`. 

If you need a snapshot it's best to use the buttons on the ANTs web page - ZIP files obtained directly from the ANTs Github page don't have version information, they are just called `ANTs-master.zip` or similar. 