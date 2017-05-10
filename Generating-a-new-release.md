# How to generate a new release

```
$ cd ${ANTsDirectory}
$ git tag v2.2.0
$ git push origin --tags
```

## Notes
* binaries can then be added by hand as they become available
    * use cpack
    * possible to automate through travis
* Release notes should be added to the Readme.md file 
