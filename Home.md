Welcome to the ANTs wiki!

See an overview of our answers to frequently asked questions and commonly requested examples [here](https://github.com/stnava/ANTsTutorial/blob/master/handout/antsGithubExamples.Rmd).

## Searching the wiki

The search bar on the right searches page titles and headings. For a full-text search of the wiki, use the main Github search bar at the top of the page, and select "in this repository". It will search for your terms in code, issues, and wiki pages.

## Installing ANTs

There are several options to install ANTs.

### Binaries

Release binaries are available for Mac, Linux, and Windows: see the [binary installation instructions](https://github.com/ANTsX/ANTs/wiki/Installing-ANTs-release-binaries). 

ANTs since 2.4.4 is available [via Conda](https://anaconda.org/aramislab/ants), thanks to Ghislain Vaillant.


### Docker

Docker images are [available on DockerHub](https://hub.docker.com/repository/docker/antsx/ants/tags).

If you are using Docker Desktop, you will probably want to increase the default RAM available to docker, see the documentation for [Mac](https://docs.docker.com/desktop/settings/mac/) [Windows](https://docs.docker.com/desktop/settings/windows/) [Linux](https://docs.docker.com/desktop/settings/linux/).


### Compiling from source

To build and install ANTs from source, see [compiling ANTs on Linux / Mac](https://github.com/ANTsX/ANTs/wiki/Compiling-ANTs-on-Linux-and-Mac-OS) or [compiling ANTs on Windows](https://github.com/ANTsX/ANTs/wiki/Compiling-ANTs-on-Windows-10). The Windows compilation instructions have not been updated in some time, and may be out of date. Please open an issue with any proposed changes.


## Using ANTs

A recommended call to `antsCorticalThickness.sh`:
```
antsCorticalThickness.sh -d $dim \
  -a $img \
  -e ${TEMPLATE_DIR}T_template0.nii.gz \
  -t ${TEMPLATE_DIR}T_template0_BrainCerebellum.nii.gz \
  -f ${TEMPLATE_DIR}T_template0_BrainCerebellumExtractionMask.nii.gz \
  -m ${TEMPLATE_DIR}T_template0_BrainCerebellumProbabilityMask.nii.gz \
  -p ${TEMPLATE_DIR}Priors/priors%d.nii.gz \
  -z 0 -x 25 -g 1 -k 1 \
  -o ${OUT_DIR}/${outname}/${outname}_
```

Fast pairwise deformable registration:

```
antsRegistrationSyNQuick.sh \
    -d 3 \
    -t s \
    -f fixed.nii.gz \
    -m moving.nii.gz \
    -o movingToFixed | tee myRegOutput.txt
```

If you run `antsRegistrationSyNQuick.sh` or `antsRegistrationSyN.sh`, it will print the full ANTs command, which you can further modify. In the above example, the command (and its output) will be stored in `myRegOutput.txt` as well as being printed to the screen.