Welcome to the ANTs wiki!

See an overview of our answers to frequently asked questions and commonly requested examples [here](https://github.com/stnava/ANTsTutorial/blob/master/handout/antsGithubExamples.Rmd).

To build and install ANTs, see [here](https://github.com/stnava/ANTs/wiki/Compiling-ANTs-on-Linux-and-Mac-OS) for Linux / Mac OS, and [here](https://github.com/stnava/ANTs/wiki/Compiling-ANTs-on-Windows-10) for Windows.

If you are installing pre-compiled binaries, start [here](https://github.com/ANTsX/ANTs/wiki/Compiling-ANTs-on-Linux-and-Mac-OS#set-path-and-antspath).

A recommended call to `antsCorticalThickness.sh`:
```
bash ${ANTSPATH}antsCorticalThickness.sh -d $dim \
  -a $img \
  -e ${TEMPLATE_DIR}T_template0.nii.gz \
  -t ${TEMPLATE_DIR}T_template0_BrainCerebellum.nii.gz \
  -f ${TEMPLATE_DIR}T_template0_BrainCerebellumExractionMask.nii.gz \
  -m ${TEMPLATE_DIR}T_template0_BrainCerebellumProbabilityMask.nii.gz \
  -p ${TEMPLATE_DIR}Priors/priors%d.nii.gz \
  -z 0 -x 25 -g 1 -k 1 \
  -o ${OUT_DIR}/${outname}/${outname}_
```

Fast pairwise deformable registration:

```
  ${ANTSPATH}antsRegistrationSyNQuick.sh \
    -d 3 \
    -t s \
    -f fixed.nii.gz \
    -m moving.nii.gz \
    -o movingToFixed | tee myRegOutput.txt
```

If you run `antsRegistrationSyNQuick.sh` or `antsRegistrationSyN.sh`, it will print the full ANTs command, which you can further modify. In the above example, the command (and its output) will be stored in `myRegOutput.txt` as well as being printed to the screen.