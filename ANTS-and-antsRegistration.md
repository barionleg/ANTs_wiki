We are frequently asked what the default `ANTS` parameters would look like in a call to `antsRegistration`. There is not a one to one mapping between the two programs, but some options can be set in a similar way.

The most frequently asked question is how to set the shrink factors `-f` and smoothing `-s` in `antsRegistration` in a way analogous to the default behavior of `ANTS`.

Example call:

```
ANTS -m MI[ fixed.nii.gz , moving.nii.gz , 1, 32] -t SyN[0.25] -r Gauss[3,0] -i 200x100x70x20 -o output
```

Where fixed and moving are T1 images with 1x1x1 mm voxels, and dimensions 160x192x256. In the deformable part, we have four levels.

For `antsRegistration`, this should be in same ballpark: `-f 6x4x2x1 -s 2x1x0.5x0 mm`. Read on for details.


## Shrink factors

Basic idea: downsampled image spacing = (input image spacing) * shrink factor

The shrink factors are set in increasing powers of two for each level, subject to a limit based on the dimensions of the image. With 4 levels, the shrink factors will be 8x4x2x1 if the image is sufficiently large in all dimensions. However, the shrinkage along each dimension is limited by the number of voxels along that dimension, such that there must be at least 32 voxels along each dimension of the downsampled image. So a 160x192x256 1x1x1 mm image would be downsampled to 5x6x8 mm spacing at the first level, 4x4x4 mm at the second level, 2x2x2 mm at the third level, and 1x1x1 mm at the fourth level.

In the case of anisotropic voxels, the shrinkage is based on the smallest voxel dimension. The downsampled image is isotropic, subject to the upper shrinkage limit based on the image size. So if the voxel dimensions of a 160x192x256 image are 0.5x0.75x1, the downsampled spacing at level 4 would be 2x2x2 mm.

Code: https://github.com/stnava/ANTs/blob/9bc1866a758c2c7b6da463566edc3cdaed65a829/ImageRegistration/itkANTSImageRegistrationOptimizer.h#L1071-L1089


## Smoothing

Smoothing is minimal in `ANTS`, based on the formula 

  sigma = (outputSpacing / inputSpacing - 1.0) * 0.2

For the example above, this translates to smoothing sigmas of 0.8x1.0x1.4 mm, 0.6x0.6x0.6 mm, 0.2x0.2x0.2 mm, and 0x0x0 mm at each level. 

Code: https://github.com/stnava/ANTs/blob/9bc1866a758c2c7b6da463566edc3cdaed65a829/ImageRegistration/itkANTSImageRegistrationOptimizer.h#L712-L748