## Input requirements

Moving image: `dt.nii.gz`
Fixed image: `fixed.nii.gz`

The DT image in this example is encoded as a symmetric matrix in the NIFTI-1 format. This can be verified by running `PrintHeader` on the image. You should see

```
    dim[0] = 5
```

followed by the x,y,z dimensions of the image, and then

```
    dim[4] = 1
    dim[5] = 6
    dim[6] = 1
    dim[7] = 1
    intent_code = 1005
```

The `intent_code` value of 1005 is the NIFTI-1 code for a symmetric matrix.

If your data is not in this format, see [Importing Diffusion Tensors](#importing-diffusion-tensors) below.

## Warp and reorient the DT

Run `antsRegistration` or `antsRegistrationSyN[Quick].sh`, registering the DT by proxy - for the moving image use B0 (recommended), FA, or some other scalar image(s) in the DT space. This produces the warps `movingDT_ToFixed1Warp.nii.gz` and `movingDT_ToFixed0GenericAffine.mat`.

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

## Importing diffusion tensors

