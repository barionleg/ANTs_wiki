Moving image: `dt.nii.gz`
Fixed image: `fixed.nii.gz`

### Compute the warp

Run `antsRegistration` or `antsRegistrationSyN[Quick].sh`, registering the DT (by proxy, eg using FA, B0, or some other scalar image in the DT space) to the fixed image. This produces the warps `movingDT_ToFixed1Warp.nii.gz` and `movingDT_ToFixed0GenericAffine.mat`.

### Apply the transform to the DT image

Apply these transforms to deform the tensor image. After this the tensors will be correctly located in the fixed space, but they retain their original orientation.

```
antsApplyTransforms -d 3 -e 2 -i dt.nii.gz -o dtDeformed.nii.gz \
-t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat -r fixed.nii.gz
```
You may combine other warps here as you would for a scalar image. For example, if the fixed image is the subject's T1, and we have transforms mapping this to template space, we can apply them to map the DT to template space. All transforms applied here, both deformable and affine, must then be composed as shown below.


### Compose the affine and deformable transforms into a single warp file for `ReorientTensorImage`

If you are doing a single affine transform only, you can proceed to the next step. 

```
antsApplyTransforms -d 3 -r fixed.nii.gz -o [dtCombinedWarp.nii.gz,1] \
-t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat \
-r fixed.nii.gz
```

### Apply the reorientation to the deformed tensor image

```
ReorientTensor 3 dtDeformed.nii.gz dtReoriented.nii.gz dtCombinedWarp.nii.gz
```