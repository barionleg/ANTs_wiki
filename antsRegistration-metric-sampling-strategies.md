Image metrics are evaluated over a collection of points in the virtual image domain. The virtual domain has the same voxel spacing and dimensions as the fixed image, after downsampling by an integer shrink factor (specified with `-f`) for the stage.

To save time, the point set can be downsampled in different ways, controlled in the metric specification. In the case of multiple metrics, the sampling strategy is defined by the first one ([code](https://github.com/ANTsX/ANTs/blob/9bc1866a758c2c7b6da463566edc3cdaed65a829/Examples/itkantsRegistrationHelper.hxx#L1284-L1309).

The different sampling strategies are explained below.


## None

Dense sampling, all points are used.


## Regular

A sampling fraction *f* is defined between 0 and 1. Every 1/_f_ points is chosen.

To reduce aliasing, a random perturbation is applied to each point. The perturbation in each dimension is drawn from a normal distribution with zero mean and variance of 1/3 the voxel spacing.


## Random

A sampling fraction *f* is defined between 0 and 1. Points are chosen at random until _f_ * (total points) have been sampled. A random perturbation is applied to the chosen points, as for regular sampling.
