# How to generate a new release

```
cd ANTs
Utilities/tagRelease.pl vX.Y.Z
```

This will update the Version.cmake and create a tag for vX.Y.Z.

The script will check 

* checked-out code is the master branch with no local modifications or unpushed commits.
* The tag does not already exist
* The tag conforms to the format vX.Y.Z, where X, Y, Z are integers. 

After running the script, go to Github and draft a new release. Auto-populate the release notes and add extra information as needed.

After publishing the release, Github Actions workflows will build binaries and update DockerHub (takes an hour or two).
