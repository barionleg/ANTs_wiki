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

The **inverse transforms** are the transforms that are used to perform the opposite operation, deforming an image in the fixed space and producing output in the moving space. This operation uses the file `movingToFixed_1InverseWarp.nii.gz` and the inverse of the forward affine transform `movingToFixed_0GenericAffine.mat`. The inverse affine transform is not usually stored because it is easy to invert on demand. However, the inverse warp field cannot be precisely computed from the forward warp field, so the inverse warp field is saved separately.


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

Internally, deforming an image involves transforming a point set. In the case of deforming the moving to the fixed image, we start from a point at the center of a voxel in fixed space, move it to the corresponding location in moving space, interpolate the intensity value at that point, and place the result in the output image. Thus the forward warps, which we use to deform an **image** from moving to fixed space, are the warps that transform **points** from fixed to moving space.





## Computing the Jacobian



## Warp naming convention in antsCorticalThickness.sh 


## Warp naming convention in antsLongitudinalCorticalThickness.sh 


## Details 

