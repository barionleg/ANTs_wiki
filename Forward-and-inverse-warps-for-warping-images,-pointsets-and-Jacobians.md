# Quick reference for applying ANTs warps

Applying the deformations computed by ANTs require the user to specify the correct warps,
and to specify them in the correct order. Getting this wrong can cause obvious or subtle
errors in the output, depending on the size of the deformations. Common use cases are
explained below.

The examples are designed to show how to apply warps, and don't necessarily represent
optimal registration parameters.

The examples do not include the `--verbose` option to `antsApplyTransforms`.
Enabling and recording verbose output is recommended to assist debugging and to clarify the
correct transform order. There are also other interpolation options besides the default
(linear) for grayscale images, see the `-n` option.


## Terminology

The command

```
antsRegistrationSyNQuick.sh
  -d 3 \
  -f fixedImage.nii.gz \
  -m movingImage.nii.gz \
  -o movingToFixed_ \
  -t s
```

produces

```
movingToFixed_1Warp.nii.gz
movingToFixed_1InverseWarp.nii.gz
movingToFixed_0GenericAffine.mat
```

The **moving** image (`-m`) and **fixed** image (`-f`) are determined when the
registration is run. It's necessary to know which is which when applying warps.

The **forward transforms** are defined as those we use to deform an image in the moving
space and produce output in the fixed space. These are `movingToFixed_1Warp.nii.gz` and
`movingToFixed_0GenericAffine.mat`.

The **inverse transforms** are used to perform the opposite operation, deforming an image
in the fixed space and producing output in the moving space. This operation uses the file
`movingToFixed_1InverseWarp.nii.gz` and the inverse of the forward affine transform
`movingToFixed_0GenericAffine.mat`. The inverse affine transform is not usually stored on
disk because it is easy to invert on demand.

It helps to have a consistent naming convention. I use an output prefix of
`movingToFixed_`, such that `movingToFixed_0GenericAffine.mat` and
`movingToFixed_1Warp.nii.gz` define the forward warp. But it's up to the user to supply
the correct warps in the correct order, `antsApplyTransforms` does not attempt to parse
transform file names for logical correctness.

When applying the warps, we will use the terms **input** (`-i`) image and **reference**
(`-r`) images, referring to `antsApplyTransforms`. For a pairwise registration, the
**input** image might be either the fixed image or the moving image, or another image in
the same physical space as either of those images. For example, we can register a T1w to a
template, then apply that warp to an ROI drawn on the T1w image, to produce a resampled
ROI in the template space.

**Important note on image vs point-set operations**: the ordering of transforms for
resampling an image into a new space is the _opposite_ of that used to transform points.
The forward warps transform a _point_ from fixed to moving space. When resampling an
image, we use the forward warps to transform a point from the center of a voxel in the
fixed image to its corresponding place in the moving image - the location of this point in
the moving image is used to determine the interpolated intensity of the voxel in the
output. When calling `antsApplyTransformsToPoints`, we are doing the same thing but
without the final step of resampling of a moving image. So we use the forward warps to
transform our points in the fixed space to points the moving space.


## Deforming an image

After a pairwise registration, like

```
antsRegistrationSyNQuick.sh
  -d 3 \
  -f fixedImage.nii.gz \
  -m movingImage.nii.gz \
  -o movingToFixed_ \
  -t s
```

Deforming the moving image to fixed space:

```
antsApplyTransforms \
  -d 3 \
  -i movingImage.nii.gz \
  -r fixedImage.nii.gz \
  -t movingToFixed_1Warp.nii.gz \
  -t movingToFixed_0GenericAffine.mat \
  -o movingToFixedDeformed.nii.gz
```

Deforming the fixed image to moving space:

```
antsApplyTransforms \
  -d 3 \
  -i fixedImage.nii.gz \
  -r movingImage.nii.gz \
  -t [ movingToFixed_0GenericAffine.mat, 1 ] \
  -t movingToFixed_1InverseWarp.nii.gz \
  -o fixedToMovingDeformed.nii.gz
```

The option `[movingToFixed_0GenericAffine.mat, 1]` tells the program to invert the affine
transform contained in `movingToFixed_0GenericAffine.mat`.

Given ROIs `fixedLabels.nii.gz`, to be resampled into the space of `movingImage.nii.gz`,
we would use the same warps:

```
antsApplyTransforms \
  -d 3 \
  -i fixedImage.nii.gz \
  -r movingImage.nii.gz \
  -t [movingToFixed_0GenericAffine.mat, 1] \
  -t movingToFixed_1InverseWarp.nii.gz \
  -n GenericLabel
  -o fixedToMovingDeformed.nii.gz
```

but we add the option `-n GenericLabel` to specify interpolation that preserves label
intensities.


## Applying affine transforms only

Calling `antsRegistration` or the registration script without a deformable transform step
will produce a `GenericAffine.mat` file but no warp field.

```
antsRegistrationSyNQuick.sh
  -d 3 \
  -f fixedImage.nii.gz \
  -m movingImage.nii.gz \
  -o movingToFixed_ \
  -t a

antsApplyTransforms \
  -d 3 \
  -i movingImage.nii.gz \
  -r fixedImage.nii.gz \
  -t movingToFixed_0GenericAffine.mat \
  -o movingToFixedAffineDeformed.nii.gz

antsApplyTransforms \
  -d 3 \
  -i fixedImage.nii.gz \
  -r movingImage.nii.gz \
  -t [movingToFixed_0GenericAffine.mat, 1] \
  -o fixedToMovingAffineDeformed.nii.gz

```

Applying the affine transform only is also useful for debugging deformable registration.
If the final result looks suboptimal, check the affine alignment and improve it if
possible, before altering the deformable parameters.


## Input image (`-i`) options

The commands above will work with any input image that is in the same **physical** space
as either the moving or fixed image. The input image can have multiple components or be a
time series, this is handled with the `-e` option. For example, if we have a BOLD time
series `bold.nii.gz` and a 3D reference image `boldRef.nii.gz` that is the first volume
from the series, we can do

```
antsRegistrationSyNQuick.sh
  -d 3 \
  -f fixedImage.nii.gz \
  -m boldRef.nii.gz \
  -o boldRefToFixed_ \
  -t r

antsApplyTransforms \
  -d 3 \
  -e 3 \
  -i bold.nii.gz \
  -r fixedImage.nii.gz \
  -t boldRefToFixed_0GenericAffine.mat \
  -o boldToFixedDeformed.nii.gz
```

to resample the time series image into the space of `fixedImage.nii.gz`.


## Reference image (`-r`) options

The reference image defines both the voxel and the physical space of the output image. The
reference image must be in the same **physical** space as the fixed image if using forward
transforms, or the moving image if using inverse transforms. If you would like to change
the resolution of the output image, you can do so by resampling the reference image with
`ResampleImageBySpacing`, which preserves the origin and orientation of the image in
physical space.

The reference image **must** have the same number of dimensions as the warp fields. For most
use cases you will run `antsApplyTransforms -d 3` and this requires a 3D reference image.
If your reference space belongs to a multi-component image, you will need a 3D image in
the same physical space. One way to obtain this is to compute an average of a 4D time
series with `antsMotionCorr`, or extracting the first 3D volume with the
`TimeSeriesSubset` function in `ImageMath`.


## Transforming a point set

Transform points from fixed to moving space:

```
antsApplyTransformsToPoints \
  -d 3 \
  -i landmarksInFixedSpace.csv \
  -o landmarksInMovingSpace.csv \
  -t movingToFixed_1Warp.nii.gz \
  -t movingToFixed_0GenericAffine.mat
```

The forward warps, which we use to deform an **image** from moving to fixed space, are the
warps that transform **points** from fixed to moving space. To move points in the opposite
direction:

```
antsApplyTransformsToPoints \
  -d 3 \
  -i landmarksInMovingSpace.csv \
  -o landmarksInFixedSpace.csv \
  -t [movingToFixed_0GenericAffine.mat, 1]
  -t movingToFixed_1InverseWarp.nii.gz \
```

The input and output to `antsApplyTransformsToPoints` is coordinates **in physical space
as defined by ITK**. This may vary from the coordinate system defined by NIFTI or other
file formats or software. Point sets should be carefully validated by users.
[ANTsR](https://github.com/ANTsX/ANTsR/wiki/MNI-Coordinates-in-ANTsR-(and-ANTs)) has some
capabilities to help with this. You can also use [ITK-SNAP](http://itksnap.org) to locate
anatomical points and look up the [ITK
coordinates](https://github.com/ANTsX/ANTs/wiki/Using-ITK-SNAP-with-ANTs#physical-space-coordinates)
interactively.


## Computing the Jacobian

```
CreateJacobianDeterminantImage 3 movingToFixed1Warp.nii.gz logJacobian.nii.gz 1 1
```

This outputs the log of the Jacobian determinant in the fixed space.

| Log Jacobian | Moving image volume change |
|    :---:     |     :---:                  |
|     < 0      | Expanding                  |
|     = 0      | None                       |
|     > 0      | Contracting                |

Simple example data and code [here](https://github.com/cookpa/jacobianExample), a more
complex example [here](https://github.com/stnava/jacobianTests).


## Warp naming convention in antsCorticalThickness.sh

In this context the moving image is the subject T1 image on which cortical thickness is
computed. The fixed image is a template.

The forward warp computed by `antsCorticalThickness.sh` is
`SubjectToTemplate1Warp.nii.gz`, and the forward affine is
`SubjectToTemplate0GenericAffine.mat`. The inverse warp is called
`TemplateToSubject0Warp.nii.gz`, and the inverse affine is saved as
`TemplateToSubject1GenericAffine.mat`, so you do not need to use the square brackets on
the command line.

To warp an image from subject to template space:

```
antsApplyTransforms \
  -d 3 \
  -i subjectImage.nii.gz \
  -r registrationTemplate.nii.gz \
  -t SubjectToTemplate1Warp.nii.gz \
  -t SubjectToTemplate0GenericAffine.mat \
  -o subjectImageToTemplateDeformed.nii.gz
```

To warp an image from template to subject space:

```
antsApplyTransforms \
  -d 3 \
  -i templateImage.nii.gz \
  -r subjectImage.nii.gz \
  -t TemplateToSubject1GenericAffine.mat \
  -t TemplateToSubject0Warp.nii.gz \
  -o templateImageToSubjectDeformed.nii.gz
```


## Warp naming convention in antsLongitudinalCorticalThickness.sh

The longitudinal pipeline contains multiple runs of `antsCorticalThickness.sh`. Together,
these provide all the transforms necessary to move any of the subject's images to the
population template space, via the intermediate single-subject template. The chain of
warps required to perform this operation in either direction is composed [by the
script](https://github.com/ANTsX/ANTs/blob/master/Scripts/antsLongitudinalCorticalThickness.sh#L1024-L1046),
and saved as `SubjectToGroupTemplateWarp.nii.gz` and `GroupTemplateToSubjectWarp.nii.gz`.
This encompasses both the affine and deformable parts.

Note that each time point will have its own warp, and images may be transformed from
subject to group template space with:

```
antsApplyTransforms \
  -d 3 \
  -i subjectImageTime1.nii.gz \
  -r groupTemplate.nii.gz \
  -t SubjectTime1/SubjectToGroupTemplateWarp.nii.gz \
  -o subjectImageTime1ToGroupTemplateDeformed.nii.gz
```

Warping to the single-subject template is similar to the cross-sectional usage:

```
antsApplyTransforms \
  -d 3 \
  -i subjectImageTime1.nii.gz \
  -r groupTemplate.nii.gz \
  -t SubjectTime1/SubjectToTemplate1Warp.nii.gz \
  -t SubjectTime1/SubjectToTemplate0GenericAffine.mat \
  -o subjectImageTime1ToGroupTemplateDeformed.nii.gz
```

## Combining warps

Wherever possible, multiple interpolations of the data should be avoided. Warps between
modalities or through intermediate templates can be combined on the command line with
multiple `-t` options to `antsApplyTransforms`.

When multiple transforms are chained together, there is scope for confusion and subtle
errors when some transforms are small. It's important to keep track of the logic of
**all** of the registrations (ie, which image is moving and which is fixed). Naming
transforms consistently helps with this.

The examples below cover some common use cases. 


## Warping multiple modalities to a common template

The usual workflow is to use the T1w or other structural image to define the most accurate
registration to a group template. The other modalities are aligned to the T1w, and the
warps are then combined to deform the other modalities to the same group template space.

By aligning to the intra-session T1w, we can exploit the fact that the underlying anatomy
is the same, and the deformations will hopefully be small. This makes the registration
problem a bit easier, though there can still be considerable challenges because of
differing contrasts and distortions. Some images may need specialized pre-processing to
correct distortions (for example, using field maps) before being aligned to the T1w.

For example, let's say we have a T1w and T2w image, which we want to align to a T1w group
template. The T2w has been acquired such that it has the same distortion characteristics as
the T1w, so any misalignment between the two is due to motion. For this example, we'll
just use a generic quick registration script to rigidly align the T2w to T1w.

```
antsRegistrationSyNQuick.sh
  -d 3 \
  -f t1.nii.gz \
  -m t2.nii.gz \
  -o t2ToT1_ \
  -t r

antsRegistrationSyNQuick.sh
  -d 3 \
  -f template.nii.gz \
  -m t1.nii.gz \
  -o t1ToTemplate_ \
  -t s
```

We can then call

```
antsApplyTransforms \
    -d 3 \
    -i t2.nii.gz \
    -o t2DeformedToTemplate.nii.gz \
    -r template.nii.gz \
    -t t1ToTemplate_1Warp.nii.gz \
    -t t1ToTemplate_0GenericAffine.mat \
    -t t2ToT1_0GenericAffine.mat
```

It may be desirable to resample the other modalities at a higher or lower resolution in the template
space, which can be done by using a resampled version of the group template as the
reference image in the call to `antsApplyTransforms`.

To warp in the other direction, eg bringing a segmentation from template to T2w space:

```
antsApplyTransforms \
    -d 3 \
    -i templateLabels.nii.gz \
    -o labelsDeformedToT2.nii.gz \
    -n GenericLabel \
    -r t2.nii.gz \
    -t [ t2ToT1_0GenericAffine.mat, 1 ] \
    -t [ t1ToTemplate_0GenericAffine.mat, 1 ] \
    -t t1ToTemplate_1InverseWarp.nii.gz
```

This is a combination of the transforms to warp the template to T1w space, and those to
warp the T1w to T2w space. Note the order here, reading right to left, starting at the
input space (template), to the intermediate space (T1w) then to the input space (T2w).

If we had used a deformable registration in the T2w to T1w registration, we would also 
include the warp field from that stage, ie


```
antsApplyTransforms \
    -d 3 \
    -i t2.nii.gz \
    -o t2DeformedToTemplate.nii.gz \
    -r template.nii.gz \
    -t t1ToTemplate_1Warp.nii.gz \
    -t t1ToTemplate_0GenericAffine.mat \
    -t t2ToT1_1Warp.nii.gz \
    -t t2ToT1_0GenericAffine.mat
```

would be used to resample the T2w image into template space, and

```
antsApplyTransforms \
    -d 3 \
    -i templateLabels.nii.gz \
    -o labelsDeformedToT2.nii.gz \
    -n GenericLabel \
    -r t2.nii.gz \
    -t [ t2ToT1_0GenericAffine.mat, 1 ] \
    -t t2ToT1_1InverseWarp.nii.gz \
    -t [ t1ToTemplate_0GenericAffine.mat, 1 ] \
    -t t1ToTemplate_1InverseWarp.nii.gz
```

would warp the template labels to the T2w space.


## Standard space through an intermediate template

Another use case is where we have a local template that has been registered to a standard
space, eg `MNI152NLin6Asym.nii.gz`. For example, say we ran

```
antsRegistrationSyN.sh
  -d 3 \
  -f MNI152NLin6Asym.nii.gz \
  -m populationTemplate.nii.gz \
  -o templateToMNI152NLin6Asym_ \
  -t s
```

And then for some subject,

```
antsRegistrationSyN.sh
  -d 3 \
  -f populationTemplate.nii.gz \
  -m t1.nii.gz \
  -o t1ToTemplate_ \
  -t s
```

Now we can warp the subject t1 to MNI space with

```
antsApplyTransforms \
    -d 3 \
    -i t1.nii.gz \
    -o t1DeformedToMNI152NLin6Asym.nii.gz \
    -r MNI152NLin6Asym.nii.gz \
    -t templateToMNI152NLin6Asym_1Warp.nii.gz \
    -t templateToMNI152NLin6Asym_0GenericAffine.mat \
    -t t1ToTemplate_1Warp.nii.gz \
    -t t1ToTemplate_0GenericAffine.mat
```

To warp some labels from MNI space to the subject image:

```
antsApplyTransforms \
    -d 3 \
    -i labels.nii.gz \
    -o labelsDeformedToT1.nii.gz \
    -n GenericLabel
    -r MNI152NLin6Asym.nii.gz \
    -t [ t1ToTemplate_0GenericAffine.mat, 1 ]
    -t t1ToTemplate_1InverseWarp.nii.gz \
    -t [ templateToMNI152NLin6Asym_0GenericAffine.mat, 1 ] \
    -t templateToMNI152NLin6Asym_1InverseWarp.nii.gz
```

## Combining forward and inverse transforms

So far, we've seen examples where we chain together transforms, but we always use either
the forward warps or the inverse. It can be helpful to design the registrations such that
the use cases work out this way, but it's not required.

Imagine we have an fMRI image `boldRef.nii.gz` but lack the field maps to correct nonlinear
distortion to T1w. We might then use `antsRegistration` to correct the distortion and
motion using the `--restrict-deformation` option. The details of this are outside the
scope of this page, but to apply the option it correctly it's necessary to make the
distorted image the **fixed** image in registration. We would then warp the T1w to boldRef
with

```
antsApplyTransforms \
  -d 3 \
  -i T1w.nii.gz \
  -r boldRef.nii.gz \
  -t t1wToBoldRef_1IWarp.nii.gz \
  -t t1wToBoldRef_0GenericAffine.mat \
  -o boldRefToT1wDeformed.nii.gz
```

and boldRef to T1w with

```
antsApplyTransforms \
  -d 3 \
  -i T1w.nii.gz \
  -r boldRef.nii.gz \
  -t [ t1wToBoldRef_0GenericAffine.mat, 1 ] \
  -t t1wToBoldRef_1InverseWarp.nii.gz \
  -o boldRefToT1wDeformed.nii.gz
```

Now if the T1w is registered to MNI152NLin6Asym via a local group template, as above, we
can resample some segmentation into the BOLD space with

```
antsApplyTransforms \
    -d 3 \
    -i labels.nii.gz \
    -o labelsDeformedToBOLD.nii.gz \
    -n GenericLabel
    -r MNI152NLin6Asym.nii.gz \
    -t boldRefToT1w_1Warp.nii.gz \
    -t boldRefToT1w_0GenericAffine.mat
    -t [ t1ToTemplate_0GenericAffine.mat, 1 ]
    -t t1ToTemplate_1InverseWarp.nii.gz \
    -t [ templateToMNI152NLin6Asym_0GenericAffine.mat, 1 ] \
    -t templateToMNI152NLin6Asym_1InverseWarp.nii.gz
```

## Collapsing warps

Users can concatenate warps manually using the `-o` option of `antsApplyTransforms`. This option produces a single warp field containing the composed transform, or a single matrix if only affine transforms are involved. 

Some warps are also collapsed by `antsRegistration` automatically, unless disabled with the `-z` option.

Affine transforms are generally performed at the beginning of the registration and collapsed into a single transform. For example:

```
antsRegistrationSyN.sh
  -d 3 \
  -f template.nii.gz \
  -m moving.nii.gz \
  -o movingToTemplate_ \
  -t a
```

will run three different stages: initial translation (with `-r`), rigid, then affine. These will be collapsed into a single transform `movingToTemplate_0GenericAffine.mat`. A user-defined initial moving transform matrix (see usage for `-r`) will also be collapsed into the combined affine transform. 

Warp fields can also be collapsed from multiple adjacent stages of deformable registration. However, a user-defined initial warp field will never be collapsed, because `antsRegistration` does not handle the initial inverse.
 

## Discussion

To verify the correct ordering of warps, it's necessary to know the choice of fixed and
moving image for every pairwise registration involved in the call to `antsApplyTransforms`.

Internally, deforming an image involves transforming a point set in the opposite direction
to the intuitive direction of the warping. When the moving image is being resampled into the
fixed space, the forward warps are used to transform a sample point set (ie, the center of
voxels in the reference image) into the moving space. Interpolated intensity values are
computed for each point and used to create the output image.

This is why the transform ordering for `antsApplyTransforms` and
`antsApplyTransformsToPoints` is reversed for a given fixed and moving image pair.