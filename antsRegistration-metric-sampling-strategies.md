Image metrics are evaluated over a collection of points in the virtual image domain. The virtual domain has the same voxel spacing and dimensions as the fixed image, after downsampling by an integer shrink factor (specified with `-f`) for the stage.

Some registration methods, such as SyN, always use dense sampling and ignore other metric options. Affine registration allows subsampling using the strategies below.


## Metric sampling strategies

To save time, the point set can be downsampled in different ways, controlled in the metric specification. In the case of multiple metrics, the sampling strategy is defined by the first one ([code](https://github.com/ANTsX/ANTs/blob/9bc1866a758c2c7b6da463566edc3cdaed65a829/Examples/itkantsRegistrationHelper.hxx#L1284-L1309)).

The different sampling strategies are explained below.


###  None

Dense sampling, all points are used.


###  Regular

A sampling fraction *f* is defined between 0 and 1. One out of every ceil(1  / _f_) points are sampled, so f = 0.5 and f = 0.6 both sample 1 out of every 2 points, *f* = 0.25 means 1 out of every 4 points is sampled, and so on.

To reduce aliasing, a random perturbation is applied to each point. The perturbation in each dimension is drawn from a normal distribution with zero mean and variance of 1/3 the voxel spacing.


###  Random

A sampling fraction *f* is defined between 0 and 1. Points are chosen at random until _f_ * (total points) have been sampled. A random perturbation is applied to the chosen points, as for regular sampling.


## Masks

With masks, the metric points are sampled in the usual way, and any points outside the mask(s) are discarded. This can lead to problems when the mask is a small fraction of the image domain. Use the initialization option (`-r` or run `antsAI`) to find an initial solution that has reasonable mask overlap.