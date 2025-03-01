Release binaries are available from ANTs 2.4.1, and Windows binaries are available from 2.4.4.


# Supported platforms

Binaries are compiled on Ubuntu (18.04, 20.04, 22.04), Centos7 (using devtoolset7), Mac OS (11 and 12) and Windows (Windows Server 2022). The builds use the default compiler for each platform.

The build is a default Superbuild. If you require custom build options (like VTK), you will need to build from source.


# Installing binaries (Mac OS and Linux)

First unzip the archive to your desired install prefix, then set the environment variables. The archives contain `ants-{version}/[bin,lib]`. To simplify the instructions, we'll omit the version numbering below and assume we have `/opt/ants/[bin,lib]`.


## Set environment variable `PATH`

Assuming your install prefix was `/opt/ants`, there will now be a binary directory `/opt/ants/bin`, containing the ANTs executables and scripts. The exact syntax to add this to the PATH may vary depending on your terminal shell. For the bash shell, you would set

```
export PATH=/opt/ants/bin:$PATH
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


### For ANTs < 2.5.0 : Also set ANTSPATH

Prior to v2.5.0, the scripts additionally require `ANTSPATH` to point to the bin directory **including a trailing slash**.

```
export ANTSPATH=/opt/ants/bin/
export PATH=${ANTSPATH}:$PATH
```

## Set `ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS` to control multi-threading at run time

Many ANTs programs use multi-threading. By default, one "worker" will be generated for every CPU core on the system. This might be acceptable on a single-user machine but in a cluster environment, you will need to restrict the number of threads to be no more than the number of cores you have reserved for use.

Set the environment variable

```
ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS
```

to control the number of threads ANTs will use. 


## Mac OS: Override security settings

Mac OS will prevent downloaded binaries from executing unless those binaries are signed. ANTs binaries are not signed because this capability requires a paid Apple Developer subscription.

To run ANTs binaries, you will need to manually add them to the list of approved programs. This is most efficiently done in the terminal with

```
spctl --add /opt/ants/bin/*
```

If this doesn't work, try

```
xattr -r -d com.apple.quarantine /opt/ants/bin/*
```

# Installing binaries (Windows)

The binaries are native Windows executables, but the scripts will require the relevant interpreters (mostly bash, some Perl). 

If any Windows users would like to contribute more detailed instructions, please open an issue with details.
