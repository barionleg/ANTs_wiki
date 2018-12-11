Warping and reorienting a diffusion tensor image is a two-stage process, because we must account for changes in orientation as well as displacement of location. 


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

[This page](https://github.com/ANTsX/ANTs/wiki/Importing-diffusion-tensor-data-from-other-software) has more information on importing diffusion tensors into ANTs.

Because tensor processing conventions vary substantially between software, we recommend testing your own data with large rotations (eg, rotate the reference image) and validating that the resulting reorientations of the tensors are correct.

**Update December 2018** - Orientation of deformed tensors was incorrect for non-axial images (where the header voxel to physical transformation matrix is not identity). See [here](https://github.com/ANTsX/ANTs/issues/642) for details. This is fixed in commit [2703896fddcf5391ca178f17ec6bc168d861156e](https://github.com/ANTsX/ANTs/commit/2703896fddcf5391ca178f17ec6bc168d861156e).


## Compute registration

Run `antsRegistration` or `antsRegistrationSyN[Quick].sh`, registering the DT by proxy - for the moving image use B0 (recommended), FA, or some other scalar image(s) in the DT space. This produces the warps `movingDT_ToFixed1Warp.nii.gz` and `movingDT_ToFixed0GenericAffine.mat`.


### Apply the transform to the DT image

Apply these transforms to deform the tensor image. After this the tensors will be correctly located in the fixed space, but they need to be reoriented to account for the rotation introduced by the registration.

```
antsApplyTransforms -d 3 -e 2 -i dt.nii.gz -o dtDeformed.nii.gz \
-t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat -r fixed.nii.gz
```

`antsApplyTransforms` will output the tensors in the space of the fixed image. 

You may combine other warps here as you would for a scalar image. For example, if the fixed image is the subject's T1, and we have transforms mapping this to template space, we can apply them to map the DT to template space. All transforms applied here, both deformable and affine, must then be composed as shown below.


### Compose the affine and deformable transforms into a single warp file for `ReorientTensorImage`

If you have computed a single affine transform only, you can reorient the tensor at this stage:

```
ReorientTensor 3 dtDeformed.nii.gz dtReoriented.nii.gz movingToFixed0GenericAffine.mat
```

Otherwise, combine the warps with `antsApplyTransforms`:

```
antsApplyTransforms -d 3 -r fixed.nii.gz -o [dtCombinedWarp.nii.gz,1] \
-t movingDT_ToFixed1Warp.nii.gz -t movingDT_ToFixed0GenericAffine.mat \
-r fixed.nii.gz
```

Then apply the reorientation to the deformed tensor image

```
ReorientTensor 3 dtDeformed.nii.gz dtReoriented.nii.gz dtCombinedWarp.nii.gz
```

The tensors are now reoriented correctly.


## Interpolation and masking options

The default linear tensor interpolation in ANTs is done in the log space. This is based on a mathematical argument that linear interpolation of the log tensor gives a better result than linear interpolation of the tensor itself, as explained here

https://www.ncbi.nlm.nih.gov/pubmed/16788917

One problem with this approach is how to interpolate background (where the DT is all zeros), since the log of 0 is undefined. By default, `antsApplyTransforms` does not modify these tensors, which are then treated as zero tensors in the log space. This is why DT images with a brain mask often have unrealistic diffusion tensors around the edge of the brain.

To prevent this, specify a background tensor diffusivity with the `-f` option to `antsApplyTransforms`. For example, `-f 0.0007` will replace background (any voxel that is all zeros) with an isotropic DT where each eigenvalue is 0.0007. If your b-values are in the usual units of s / mm^2 , then this ought to create a DT with similar mean diffusivity to those in the brain GM / WM. If your b-values are in different units, scale accordingly. Alternatively, you could use a larger value to simulate interpolation with CSF, which presumably surrounds the brain.

Another option is to mask the DT image more generously in the native space, and then apply a tighter brain mask after warping to structural space, so that the voxels interpolated with zero voxels will be removed. Lastly, nearest-neighbor interpolation avoids this issue entirely, though results may be less accurate within the brain.