# Quick reference for applying ANTs warps

Applying the deformations computed by ANTs require the user to specify the correct warps, and to specify them in the correct order. Common use cases are explained below.

## Terminology

The command

```
${ANTSPATH}antsRegistrationSyNQuick.sh 
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

The **forward transforms** from moving to fixed space are defined as those we use to deform an image in the moving space and produce output in the fixed space. These are `movingToFixed_1Warp.nii.gz` and `movingToFixed_0GenericAffine.mat`.

The **inverse transforms** are the transforms that are used to perform the opposite operation, deforming an image in the fixed space and producing output in the moving space. This operation uses the file `movingToFixed_1InverseWarp.nii.gz` and the inverse of the forward affine transform `movingToFixed_0GenericAffine.mat`. The inverse affine transform is not usually stored because it is easy to invert on demand (however, see the section on `antsCorticalThickness.sh` below). The inverse warp field cannot be precisely computed from the forward warp field, so the inverse warp field is saved separately.


## Deforming an image

Deforming the moving image to fixed space:

```
${ANTSPATH}antsApplyTransforms \
  -d 3 \
  -i movingImage.nii.gz \   
  -r fixedImage.nii.gz \   
  -t movingToFixed_1Warp.nii.gz \
  -t movingToFixed_0GenericAffine.mat \   
  -o movingToFixedDeformed.nii.gz
```

Regions of interest are contained in `fixedLabels.nii.gz`, in the fixed image space. Deforming these to moving space:

```
${ANTSPATH}antsApplyTransforms \
  -d 3 \
  -i fixedLabels.nii.gz \
  -r movingImage.nii.gz \   
  -t [movingToFixed_0GenericAffine.mat, 1] \   
  -t movingToFixed_1InverseWarp.nii.gz \
  -n GenericLabel[Linear] \
  -o movingToFixedDeformed.nii.gz
```

The option `[movingToFixed_0GenericAffine.mat, 1]` tells the program to invert the affine transform contained in `movingToFixed_0GenericAffine.mat`. 


## Transforming a point set

Transform points from fixed to moving space:

```
${ANTSPATH}antsApplyTransformsToPoints \
  -d 3 \
  -i landmarksInFixedSpace.csv \
  -o landmarksInMovingSpace.csv \
  -t movingToFixed_1Warp.nii.gz \
  -t movingToFixed_0GenericAffine.mat 
```

The forward warps, which we use to deform an **image** from moving to fixed space, are the warps that transform **points** from fixed to moving space. To move points in the opposite direction:

```
${ANTSPATH}antsApplyTransformsToPoints \
  -d 3 \
  -i landmarksInMovingSpace.csv \
  -o landmarksInFixedSpace.csv \
  -t [movingToFixed_0GenericAffine.mat, 1]
  -t movingToFixed_1InverseWarp.nii.gz \
```

The input and output to `antsApplyTransformsToPoints` is in physical space as defined by ITK. This may vary from the coordinates as understood by NIFTI or other file formats. The coordinates need to be carefully validated by users. [ANTsR](https://github.com/ANTsX/ANTsR/wiki/MNI-Coordinates-in-ANTsR-(and-ANTs)) has some capabilities to help with this.


## Computing the Jacobian

The Jacobian is typically computed in the fixed space, so that Jacobians from a population of moving images can compared directly.

```
CreateJacobianDeterminantImage 3 movingToFixed1Warp.nii.gz logJacobian.nii.gz 1 1
```

This outputs the log of the Jacobian determinant in the fixed space. 

| Log Jacobian | Moving image volume change | 
|    :---:     |     :---:                  | 
|     < 0      | Expanding                  | 
|     = 0      | Zero                       |
|     > 0      | Contracting                |
 
Simple example data and code [here](https://github.com/cookpa/jacobianExample), a more complex example [here](https://github.com/stnava/jacobianTests).


## Warp naming convention in antsCorticalThickness.sh 

The forward warp computed by `antsCorticalThickness.sh` is `SubjectToTemplate1Warp.nii.gz`, and the forward affine is `SubjectToTemplate0GenericAffine.mat`. The inverse warp is called `TemplateToSubject0Warp.nii.gz`, and the inverse affine is saved as `TemplateToSubject1GenericAffine.mat`, so you do not need to use the square brackets on the command line.

To warp an image from subject to template space:

```
${ANTSPATH}antsApplyTransforms \
  -d 3 \
  -i subjectImage.nii.gz \
  -r registrationTemplate.nii.gz \   
  -t SubjectToTemplate1Warp.nii.gz \
  -t SubjectToTemplate0GenericAffine.mat \
  -o subjectImageToTemplateDeformed.nii.gz
```
 
To warp an image from template to subject space:

```
${ANTSPATH}antsApplyTransforms \
  -d 3 \
  -i templateImage.nii.gz \
  -r subjectImage.nii.gz \   
  -t TemplateToSubject1GenericAffine.mat \
  -t TemplateToSubject0Warp.nii.gz \
  -o templateImageToSubjectDeformed.nii.gz
```


## Warp naming convention in antsLongitudinalCorticalThickness.sh 


## Details 

Internally, deforming an image involves transforming a point set in the opposite direction to the intuitive direction of the warping. The "moving" image appears to be moving, but in reality it's being resampled. The sample points are a regular grid of voxel centers in the fixed space. A point at the center of a voxel is transformed to the corresponding location in moving space, an interpolated intensity value is computed, and the result is placed in the voxel in the output image. 

This is why the transform ordering for `antsApplyTransforms` and `antsApplyTransformsToPoints` is different, and the use of the forward warp for the Jacobian may be counter-intuitive.
