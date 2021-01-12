# Quick start

Example call for human brain T1w:

```
N4BiasFieldCorrection -d 3 -v 1 -s 4 -b [ 180 ] -c [ 50x50x50x50, 0.0 ] \
  -i T1w.nii.gz -o [ corrected_T1w.nii.gz, T1w_BiasField.nii.gz ]
```

If you have multi-component or time series data, do not use `-d 4`. Compute the bias field on a representative 3D volume and apply it by dividing each component or measurement by the bias field. More on this below.


# Algorithm

From the program usage: 

>N4 is a variant of the popular N3 (nonparameteric nonuniform normalization) retrospective bias correction algorithm. Based on the assumption that the corruption of the low frequency bias field can be modeled as a convolution of the intensity histogram by a Gaussian, the basic algorithmic protocol is to iterate between deconvolving the intensity histogram by a Gaussian, remapping the intensities, and then spatially smoothing this result by a B-spline modeling of the bias field itself. 

The original [N3 paper](https://pubmed.ncbi.nlm.nih.gov/9617910/) explains the fundamentals in detail. The [N4 paper](https://pubmed.ncbi.nlm.nih.gov/20378467/) explains the improvements and additions that have been implemented in ANTs.


# Basic usage options

The default parameters are optimized for correcting human brain T1w images. The initial spline spacing should always be set explicitly. For human brain images, this should generally be somewhere between 100 and 200 mm.


## Spline spacing (-b)

The spline spacing can be set as a single number in mm, for isotropic spacing, or as a mesh resolution in each dimension. The image will be padded to fit an integer number of spacings into the image domain. Smaller spacing allows the bias field to be more detailed. 

The spacing parameter interacts with the convergence parameter: the spline spacing is doubled at each level, so `-b [ 100 ] -c [ 50x20, 0.0 ]` runs 50 iterations with 100mm spacing, then 20 with 50mm spacing.

The second parameter to `-b` is the spline order. The default is 3, so `-b [ 180 ]` is equivalent to `-b [ 180, 3 ]`, which is sufficient for most uses.



## Convergence (-c)

The convergence option allows the user to set the number of multi-resolution iterations, and a convergence parameter which can halt execution of each level. The number of iterations and levels should be chosen to find a good solution without overfitting or taking too much time. 


## Shrink factor (-s)

The input image is downsampled by an integer shrink factor to speed up processing. The default is 4. The corrected image and bias field are output at full resolution.


## Masking (-x, -w)

Masks restrict the voxels in which the image histogram is sampled. The `-x` option requires a binary mask. Only voxels inside the mask are corrected in the output image. If this is not desired, use `-w`. The `-w` option takes a weight image which does not have to be binary. For example, a smoothed brain mask, or tissue probabilities.


# Usage for time-series or multi-component data

Using `-d 4` fits a 4D, or time-varying, bias field. In most cases, it is much better to determine a static bias field on a representative 3D image. This is commonly one to improve inter-modality registration.

If you want to correct the time series itself, you can save the 3D bias field and apply it to all volumes / components of the data set. The bias-corrected image is the input image divided by the bias field.


## Averaging time series data

The average image from a time series can be computed with `antsMotionCorr`. 


## Averaging dMRI data

Because diffusion images have strongly varying contrast between gradients, it's best to use either the first unweighted (b=0) volume or the average of several b=0 volumes.

N4 is used by the `dwibiascorrect` program in [mrtrix](https://mrtrix.readthedocs.io/en/latest/reference/commands/dwibiascorrect.html). The default parameters are `-b [ 100, 3 ] -c [ 1000, 0.0 ] -s 4`. This fits a relatively smooth bias field compared to the N4 defaults for structural images.


# Troubleshooting

## Intensity outliers and negative values

Because the bias correction operates in the log domain, the input images need to be positive. Non-positive background values can be removed from the computation by using a mask. 

Outlier intensities can also cause problems for the intensity histogram. Winsorizing these can improve performance.


## Out of memory errors

This usually happens in HPC environments where memory usage is limited. Padding of the image does increase memory use, but it is usually only a problem if the image is of very high resolution, or the spline distance is set inappropriately (eg, using `-b [ 200 ]` on a mouse brain).


## Wrong dimensionality

Using `-d 4` to compute a bias field is likely to lead to strange results unless you really want a time-varying bias field and can optimize the spline parameters in all four dimensions. Likewise, for a time series of 2D images, use `-d 2` and apply the bias field to each slice.