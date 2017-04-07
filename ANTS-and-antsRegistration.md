We are frequently asked what the default `ANTS` parameters would look like in a call to `antsRegistration`. There is not a one to one mapping between the two programs, but some options can be set in a similar way.

The most frequently asked question is how to set the shrink factors `-f` and smoothing `-s` in `antsRegistration` in a way analogous to the default behavior of `ANTS`. This page explains how ANTS handles these parameters for non-rigid registration. 

Example call:

```
ANTS -m MI[ fixed.nii.gz , moving.nii.gz , 1, 32] -t SyN[0.25] -r Gauss[3,0] -i 200x100x70x20 -o output
```

Where fixed and moving are T1 images with 1x1x1 mm voxels, and dimensions 160x192x256. In the deformable part, we have four levels.

For `antsRegistration`, this should be in same ballpark: `-f 6x4x2x1 -s 2x1x0.5x0 mm`. 


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


## Affine registration

The default call to `ANTS` sets the following parameters:

```
--affine-metric-type MI
--MI-option 32x32000
--number-of-affine-iterations 10000x10000x10000
```

This does an initial affine transform using three levels, with the MI metric. The MI metric in `antsRegistration` is a synonym for Mattes Mutual Information, a different implementation of MI to that used in `ANTS`. 

The downsampling and smoothing parameters are the same as described above for deformable registration, so something like 1-c [10000x10000x10000, 1e-4, 10] -f 4x2x1 -s 0.6x0.2x0mm` for a typical 1mm brain image. The number of MI bins is set to 32, but in contrast to `antsRegistration`, a fixed number of sample points are specified (32000). In `antsRegistration`, we set a sampling percentage rather than a fixed number of samples. For example, 

```
  -m MI[fixed.nii.gz, moving.nii.gz, 1,32, Regular, 0.1]
```

samples points on a regular grid, such that 10% of the voxels contain a sample point. Replacing "Regular" with "Random" adds a random perturbation to each point, to minimize any bias arising from regular sampling.

The default number of MI samples in `ANTS` translates to a different sampling percentage for different images, so there's no direct replacement for this value. The setting in the `antsRegistrationSyN.sh` script is 0.25. Sampling more densely improves the capture range of the registration, at the cost of computation time. With a good initialization it may be possible to reduce the sampling percentage without harming performance.

We have also found it useful to start with a rigid alignment before doing an affine stage. In ANTS, setting `--do-rigid` does a rigid alignment instead of, rather than before, affine.

