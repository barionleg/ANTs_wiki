Warping and reorienting a diffusion tensor image is a two-stage process, because we must account for changes in orientation as well as displacement of location. Because tensor processing conventions vary substantially between software, we recommend testing your own data with large rotations (eg, rotate the reference image) and validating that the resulting reorientations of the tensors are correct.

ANTs can read and write many file types supported by ITK, but for simplicity this page assumes diffusion tensors stored in NIFTI-1 format. 

For all file types, two key things to check are correct component ordering of the tensor for that file type, and that the orientation of the tensors can be mapped to the physical space of the image using the rigid transform that ITK reads from the header.


# Input requirements for NIFTI-1 images

Moving image: `dt.nii.gz`
Fixed image: `fixed.nii.gz`

The diffusion tensors must follow the specifications outlined below. See the wiki page for [Importing diffusion tensor data from other software](https://github.com/ANTsX/ANTs/wiki/Importing-diffusion-tensor-data-from-other-software) for examples of how to import tensors from FSL and other software.


## Header requirements

The DT image should be encoded as a symmetric matrix in the NIFTI-1 format. This can be verified by running `PrintHeader` on the image. You should see

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


## Component ordering

ITK expects tensors in NIFTI format to follow the NIFTI specification of **lower-triangular** ordering: [dxx, dxy, dyy, dxz, dyz, dzz]. 


## Diffusion tensor orientation 

For ANTs to handle the tensor orientation correctly, the tensors must be oriented in the voxel space of the image, such that they can be converted to physical space using the ITK direction matrix. 

The command `RebaseTensorImage` can be used to reorient tensors stored in physical space to voxel space.

To check orientation, use `ImageMath 3 rgb.nii.gz TensorColor dt.nii.gz`. This will produce an RGB image based on the tensor coordinate system. The colors may change after calling `RebaseTensorImage`, depending on how the data is stored.


# Compute registration

Run `antsRegistration` or `antsRegistrationSyN[Quick].sh`, registering the DT by proxy - for the moving image use B0 (recommended), FA, or some other scalar image(s) in the DT space. This produces the warps `movingDT_ToFixed1Warp.nii.gz` and `movingDT_ToFixed0GenericAffine.mat`, or just `movingDT_ToFixed0GenericAffine.mat` if the transformation does not include deformable registration.


# Apply the transform to the DT image

`antsApplyTransforms` will warp the tensors with the `-e 2` option. After this, the tensors will be correctly **located** in the fixed space, but they will retain their original **orientation** - they need to be reoriented to account for the rotation introduced by the registration.

```
antsApplyTransforms -d 3 -e 2 -i dt.nii.gz -o dtDeformed.nii.gz \
  -t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat -r fixed.nii.gz
```

You may combine other warps here as you would for a scalar image. For example, if the fixed image is the subject's T1, and we have transforms mapping this to template space, we can apply them to map the DT to template space. All transforms applied here, both deformable and affine, must be composed as shown below for reorientation.


# Compose the affine and deformable transforms into a single warp file for `ReorientTensorImage`

If you have computed a single affine transform only, you can reorient the tensor at this stage:

```
ReorientTensorImage 3 dtDeformed.nii.gz dtReoriented.nii.gz movingToFixed0GenericAffine.mat
```

Otherwise, combine the warps with `antsApplyTransforms`:

```
antsApplyTransforms -d 3 -o [dtCombinedWarp.nii.gz,1] \
  -t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat \
  -r fixed.nii.gz
```

Then apply the reorientation to the deformed tensor image

```
ReorientTensor 3 dtDeformed.nii.gz dtReoriented.nii.gz dtCombinedWarp.nii.gz
```

The tensors are now reoriented correctly.


## Combining inter-subject warps

The above example uses the output of a single registration, which is normally done to align the DWI space to the T1w structural image acquired in the same session. Additional warps can be composed and applied to align the DT to a template. If we have aligned the DT to an anatomical image with transform `diffusionToAnat0GenericAffine.mat` and the anatomical to a group template with `anatToGroupTemplate1Warp.nii.gz anatToGroupTemplate0GenericAffine.mat`, then these transforms need to be composed to reorient the DT to the group template space. The transform syntax is the same as for scalar images, the only difference is we use `-e 2` to deform the tensor image, and then apply the reorientation step.

```
antsApplyTransforms -d 3 -i dt.nii.gz -o dtGroupTemplateDeformed.nii.gz -e 2 \
  -t anatToGroupTemplate1Warp.nii.gz -t anatToGroupTemplate0GenericAffine.mat \
  -t diffusionToAnat0GenericAffine.mat \
  -r groupTemplate.nii.gz 

antsApplyTransforms -d 3 -o [dtCombinedWarp.nii.gz,1] \
  -t anatToGroupTemplate1Warp.nii.gz -t anatToGroupTemplate0GenericAffine.mat \
  -t diffusionToAnat0GenericAffine.mat \
  -r groupTemplate.nii.gz 

ReorientTensorImage 3 dtGroupTemplateDeformed.nii.gz \
  dtNormalizedToGroupTemplate.nii.gz dtCombinedWarp.nii.gz
```


# Interpolation and masking options

The default linear tensor interpolation in ANTs is done in the log space. This is based on a mathematical argument that linear interpolation of the log tensor gives a better result than linear interpolation of the tensor itself, as explained here

https://www.ncbi.nlm.nih.gov/pubmed/16788917

One problem with this approach is how to interpolate background (where the DT is all zeros), since the log of 0 is undefined. By default, `antsApplyTransforms` does not modify these tensors, which are then treated as zero tensors in the log space. This is why DT images with a brain mask often have unrealistic diffusion tensors around the edge of the brain.

To prevent this, specify a background tensor diffusivity with the `-f` option to `antsApplyTransforms`. For example, `-f 0.0007` will replace background (any voxel that is all zeros) with an isotropic DT where each eigenvalue is 0.0007. If your b-values are in the usual units of s / mm^2 , then this ought to create a DT with similar mean diffusivity to those in the brain GM / WM. If your b-values are in different units, scale accordingly. Alternatively, you could use a larger value to simulate interpolation with CSF, which presumably surrounds the brain.

Another option is to mask the DT image more generously in the native space, and then apply a tighter brain mask after warping to structural space, so that the voxels interpolated with zero voxels will be removed. Lastly, nearest-neighbor interpolation avoids this issue entirely, though results may be less accurate within the brain.


# Troubleshooting

## Check input tensors

To check the correctness of the tensors in ITK format, use ImageMath and [ITK-SNAP](https://github.com/ANTsX/ANTs/wiki/Using-ITK-SNAP-with-ANTs) for visualization:

```
ImageMath dtFA.nii.gz TensorFA dt.nii.gz
ImageMath dtMD.nii.gz TensorMeanDiffusion dt.nii.gz
ImageMath dtRGB.nii.gz TensorColor dt.nii.gz
itksnap -g dtFA.nii.gz -o dtMD.nii.gz dtRGB.nii.gz
```

If the FA or MD image looks wrong, it is often because the tensors are not stored in the correct component order on disk.

As well as checking the correctness of the scalar values of FA and MD, you should also check that the orientation (particularly left-right) is correct.


## Input tensors are correct; deformed image is misaligned

If the registration is poor or the warps are incorrectly applied, the alignment of the deformed image will be incorrect. Troubleshooting this step can be made easier by using a scalar image in the same space as the diffusion tensors. Check that the registration and call to `antsApplyTransforms` is correct by applying the transforms to the scalar image that were input to the registration (eg, the b=0 volume). [This page](https://github.com/ANTsX/ANTs/wiki/Forward-and-inverse-warps-for-warping-images,-pointsets-and-Jacobians) has more information on ordering and combining warps.


## Deformed image is aligned but the tensor scalars are wrong 

Interpolation artifacts (see above) can cause problems with visualization of the mean diffusivity by introducing outliers around the edge of the brain. Try warping the tensors with `-n NearestNeighbor`.


## Deformed tensor scalars are correct but the reorientation is wrong

Scalar metrics like fractional anisotropy and mean diffusivity are rotationally invariant, meaning they don't change if you apply a rotation to the tensor. So it's possible for FA to be correct but for the tensor orientations to be wrong. This is a difficult issue to solve because it often involves re-exporting the final image to other software that performs tractography.

ANTs uses the ITK direction cosine matrix to transform tensors on disk to physical space. Therefore, reorientation will only work if the b-vectors used to fit the tensor can be rotated to physical space using the image header transform. Example data to test this is available [here](https://github.com/cookpa/antsDTOrientationTests).