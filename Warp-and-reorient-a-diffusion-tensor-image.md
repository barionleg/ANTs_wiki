Moving image: `dt.nii.gz`
Fixed image: `fixed.nii.gz`

1.   Run `antsRegistration` or `antsRegistrationSyN[quick].sh`, registering the DT (by proxy, eg using FA, B0, or some other scalar image in the DT space) to the fixed image. This produces the warps `movingDT_ToFixed1Warp.nii.gz` and `movingDT_ToFixed0GenericAffine.mat`.

2. Apply these transforms to deform the tensor image. After this the tensors will be correctly located in the fixed space, but they retain their original orientation.

```
antsApplyTransforms -d 3 -e 2 -r fixed.nii.gz -i dt.nii.gz \
-o dtDeformed.nii.gz -t movingDT_ToFixed1Warp.nii.gz \
-t movingDT_ToFixed0GenericAffine.mat -r fixed.nii.gz
```

3. Compose the warps into a single warp file for `ReorientTensorImage`

```
antsApplyTransforms -d 3 -r fixed.nii.gz -o [dtCombinedWarp.nii.gz,1] \
-t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat \
-r fixed.nii.gz
```

4. Apply the reorientation to the deformed tensor image

```
ReorientTensor 3 dtDeformed.nii.gz dtReoriented.nii.gz \
dtCombinedWarp.nii.gz
```