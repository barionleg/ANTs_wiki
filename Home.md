Welcome to the ANTs wiki!


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