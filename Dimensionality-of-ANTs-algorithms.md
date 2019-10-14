In general, the `-d` flag controls the dimensionality of the algorithm being used, but this can be confusing in some use cases, such as when 3D registration warps are applied to each volume in a 4D time series. 

It is very important to choose the dimensionality correctly and to ensure that all input and reference images have the expected dimensions. If there is a discrepency, the output physical space may not be correct (see [here](https://github.com/stnava/ANTs/issues/250) for details).

## Use of the -d flag in ANTs programs

### `antsRegistration` 

Supported dimensions: 2/3/4

The dimension chosen here determines the domain of the registration. With `-d 4`, the registration is done across all four dimensions. This allows the user to pass in two time series and compute the registration over time as well as space. 

The more common use case is where a 4D time series (such as fMRI data) is registered to a 3D image (such as a T1w volume). In this case, you would use `-d 3` and compute a 3D moving image, such as the mean fMRI volume.


### `antsApplyTransforms`

Supported dimensions: 2/3/4

To apply a 3D warp (or multiple 3D warps) to each volume of a multi-component image or time series, use `-d 3` and the `-e` option. Only use `-d 4` if you are supplying 4D warp(s).

The reference image must also match the dimension of the warp. Usually, the reference image will be one of the images you used in `antsRegistration`.


### `N4BiasFieldCorrection`

Supported dimensions: 2/3/4

Using `-d 4` will compute a time-varying bias field. As with registration, you probably want to define a 3D bias field on the first volume or maybe on an average, and then apply it to each volume of the 4D data. See the `-o` option for how to output the bias field. `ImageMath` has a command `TimeSeriesDisassemble` to convert a 4D volume into a series of 3D images.