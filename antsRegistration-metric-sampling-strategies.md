Image metrics are evaluated over a collection of points in the virtual image domain. The virtual domain has the same voxel spacing and dimensions as the fixed image, after downsampling by an integer shrink factor (specified with `-f`) for the stage.

If the image spacing is anisotropic, the shrink factor is applied to the smallest spacing, shrink factors are chosen along the other dimensions in order to make the resulting point set close to isotropic ([code](https://github.com/ANTsX/ANTs/blob/14e7312928179387e74a941f4b48223ffb9f4052/Examples/itkantsRegistrationHelper.hxx#L584-L632)). For example, given an image with 0.5x1x2mm spacing, `-f 8` would result in shrink factors of [8,4,2], and the resulting point set would have spacing 4x4x4mm.

Prior to v2.4.3, SyN registration always used dense sampling, regardless of the metric options. It now supports regular and random sampling.

## Metric sampling strategies

To save time, the point set can be downsampled in different ways, controlled in the metric specification. In the case of multiple metrics, the sampling strategy is defined by the first one ([code](https://github.com/ANTsX/ANTs/blob/9bc1866a758c2c7b6da463566edc3cdaed65a829/Examples/itkantsRegistrationHelper.hxx#L1284-L1309)).

The different sampling strategies are explained below.


###  None

Dense sampling, all points are used.


###  Regular

A sampling fraction *f* is defined between 0 and 1. One out of every ceil(1  / _f_) points are sampled, so *f* = 0.5 and *f* = 0.6 both sample 1 out of every 2 points, *f* = 0.25 means 1 out of every 4 points is sampled, and so on.

To reduce aliasing, a random perturbation is applied to each point. The perturbation in each dimension is drawn from a normal distribution with zero mean and standard deviation of 1/3 the voxel spacing ([code](https://github.com/InsightSoftwareConsortium/ITK/blob/0539a2c4ddd2b189d1e48eaf5294ce5556efe732/Modules/Registration/RegistrationMethodsv4/include/itkImageRegistrationMethodv4.hxx#L992-L1021)).


###  Random

A sampling fraction *f* is defined between 0 and 1. Points are chosen at random from the dense grid of *N* points until *fN* points have been sampled. A random perturbation is applied to the chosen points, as for regular sampling ([code](https://github.com/InsightSoftwareConsortium/ITK/blob/0539a2c4ddd2b189d1e48eaf5294ce5556efe732/Modules/Registration/RegistrationMethodsv4/include/itkImageRegistrationMethodv4.hxx#L1022-L1055))

## Masks

With masks, the metric points are sampled in the usual way, and any points outside the mask(s) are discarded. This can lead to problems when the mask is a small fraction of the image domain. Use the initialization option (`-r` or run `antsAI`) to find an initial solution that has reasonable mask overlap.