The `-d` flag, or for older programs the "ImageDimension" positional argument, controls the dimensionality of the algorithm being used, but this can be confusing in some use cases, such as when 3D registration warps are applied to each volume in a 4D time series. 

Dimensionality / ImageDimension means the number of dimensions that are defined for the data. This is distinct from the size of the image. An image of 128x128x128 voxels and one of 256x256x256 voxels have the same dimensionality (3), though they are a different size. 

The user must choose the appropriate dimensionality and ensure that all input and reference images have the expected dimensionality. If there is a discrepancy, the output physical space may not be correct (see [here](https://github.com/stnava/ANTs/issues/250) for details).


# Use of the -d flag in registration programs

In general, The dimensionality matches the registration problem being solved, or the warps being applied. This can differ from the dimensionality of the input data.


## `antsMotionCorr`

Supported dimensions: 2/3

To correct a series of 3D volumes, use `-d 3`. The correction is performed on pairs of 3D volumes, so we use `-d 3` even though the input is a 4D image.


## `antsRegistration` 

Supported dimensions: 2/3/4

Both the moving and fixed images must have the same dimensionality. 

The dimension chosen here determines the domain of the registration. With `-d 4`, the registration is done across all four dimensions. This allows the user to pass in two time series and compute the registration over time as well as space. 

The more common use case is where a 4D time series (such as fMRI data) is registered to a 3D image (such as a T1w volume). In this case, you would compute a 3D moving image, such as the mean fMRI volume, and then run the registration with `-d 3`. This computes a single 3D warp from the fMRI space to the T1w space, which can be applied to the 4D time series with `antsApplyTransforms`.


## `antsApplyTransforms`

Supported dimensions: 2/3/4

To apply 3D warps to each volume of a multi-component image or time series, use `-d 3` and the `-e` option. Only use `-d 4` if you are supplying 4D warp(s).

The reference image must match the dimension of the warp. Usually, the reference image will be one of the images you used in `antsRegistration`.

In other words, if you ran `antsRegistration -d N`, you must call `antsApplyTransforms -d N -r refImage` and `refImage` must be an N-dimensional image.

### `antsApplyTransforms -e`

Multi-component and time-series data can be warped with `antsApplyTransforms`. These data are handled with the `-e` option. 

`-d N` means the dimensionality of the warp, and implies an N-dimensional grid of N-dimensional warp vectors. This matches the `-d` in the call to `antsRegistration`. `-e M` means the type of moving image. 

* `-e 0` (scalar) is an N-dimensional grid of scalars. This is the default.
* `-e 1` (vector) is an N-dimensional grid of N-dimensional vectors. 
* `-e 2` (tensor) is an N-dimensional grid of N-dimensional tensors. 
* `-e 3` (time series) is an (N+1)-dimensional grid of scalars, the warp of dimension N is applied to each volume.
* `-e 4` (multi-component) is an N-dimensional grid, with an arbitrary number of independent scalar components at each point. The warp is applied to each scalar independently.


# Use of the -d flag in other programs

## `N4BiasFieldCorrection`

Supported dimensions: 2/3/4

Using `-d 4` will compute a time-varying bias field. As with registration, you probably want to define a 3D bias field on the first volume or on an average, then apply it to each volume of the 4D data. See the `-o` option for how to output the bias field. `ImageMath` has a command `TimeSeriesDisassemble` to convert a 4D volume into a series of 3D images, you can them recombine them with `TimeSeriesAssemble`. Divide each 3D volume by the bias field to correct the series.

