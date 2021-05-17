# __Problem:__  multiple runs of `antsRegistration` on the same images produces different results

Variance in output across multiple runs of the same registrations is a result of random sampling and floating point precision errors, as discussed below. These variances are usually small, and reflect uncertainty inherent to the registration method and to computation generally. We caution that a reproducible solution is not necessarily more accurate. However, some applications require complete reproducibility.

To meet this requirement, a "repro mode" has been added to the `antsRegistrationSyN.sh` and `antsRegistrationSyNQuick.sh`. This uses a fixed seed for the random number generation and uses reproducible metrics for registration. This should provide complete reproducibility, at the cost of computational time and possibly registration accuracy. Alternatively, a fixed seed and single-threaded execution will produce reproducible results.


# Causes of variance

## Floating point precision errors

In all scientific computing, the limited precision of floating point operations introduces variance that can be mitigated (often at the cost of performance), but usually not eliminated. Strategies such as [compensated summation](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) exist to manage precision errors.  

Floating point precision may cause differences in the registration solution on the same images in several contexts including

* Multi-threading. The exact sequence of floating point operations depends on the number of threads, as the results of computations performed in parallel are combined. This affects global metrics (like mutual information) but not local metrics (like cross-correlation).

* User choice of single or double precision for floating point operations.

* Differences in CPU architecture.

* Differences in compiler options for ANTs, ITK, or underlying system libraries.


## Random sampling

In registration, the point set used to evaluate similarity metrics is initialized as the grid of voxel centers of the fixed image. The point set is then [perturbed randomly](https://github.com/InsightSoftwareConsortium/ITK/blob/master/Modules/Registration/RegistrationMethodsv4/include/itkImageRegistrationMethodv4.hxx#L917-L1076) to [reduce bias in estimation](http://bigwww.epfl.ch/preprints/thevenaz0602p.pdf). This appears to be the largest source of variance in registration results, according to the experiments described below. 


# Strategies to improve reproducibility

## Repro mode

Repro mode uses a fixed seed, and uses global correlation (GC) instead of mutual information for linear registration. While this is slower and possibly less robust than mutual information (MI), it is reproducible with multiple threads. 

In `antsRegistrationSyNQuick.sh`, repro mode additionally changes the deformable metric from the default (MI) to cross correlation (CC). This will also increase computation time, which again can be mitigated by multi-threading.


## Alternatives to repro mode

* Set a fixed seed for randomization on the command line or by exporting the variable `ANTS_RANDOM_SEED`.

* Disable multi-threading if using global metrics such as mutual information.

These two steps should provide reproducibility at the cost of increased computation time. 


# Quantification of variance in registration results

The variance in a simple registration task is mostly due to random point set sampling / perturbation. This can be removed by dense sampling or use of a fixed seed. Multi-threading and single vs double precision appear to have a small impact on average.

This was evaluated by running `antsRegistrationSyNQuick.sh` repeatedly on a set of 10 brains, and computing the pairwise overlap of brain labels between the different runs. Specifically, `antsRegistrationSyNQuick.sh` was called 25 times for each of 10 brain images, registering to a common template. The mean (over all subjects) Dice overlap of the cerebral white matter under different experimental conditions is listed below. Scripts and results for other brain regions are available [here](https://github.com/cookpa/antsRegReproduce).

| Fixed Random Seed | Float precision | Threads | Mean Dice | SD Dice |
| --- | --- | --- | --- | --- |
| TRUE  | double | 1 | 1.000 | 0.000 |
| TRUE  | single | 1 | 1.000 | 0.000 |
| TRUE  | double | 2 | 0.993 | 0.005 |
| FALSE | double | 1 | 0.978 | 0.009 |
| FALSE | single | 1 | 0.979 | 0.008 |
| FALSE | single | 2 | 0.978 | 0.009 |


## Limitations of these experiments

 * Quick registration, may be less stable than `antsRegistrationSyN.sh` (but much faster). 

 * Random sampling only affects the rigid and affine stages of the registration, because dense sampling is used for SyN stage. This is also the case for `antsRegistrationSyN.sh` and `antsCorticalThickness.sh`. Using random sampling in the SyN stage would likely increase variance.

 * Overlap between runs measures reproducibility, not registration quality, so it does not address the question of whether using random sampling or double precision improves registration overall. 


# Related discussion threads

* [Beginnings of a default set up for reproducible registration in antsx](https://github.com/ANTsX/ANTs/issues/1189)

* [antsRegistration does not produce equivalent results](https://github.com/ANTsX/ANTsR/issues/210#issuecomment-377511054)

* [Non-deterministic result of antsAffineInitializer](https://github.com/ANTsX/ANTs/issues/444)

* [Fix registration ANTs random seed with an environment variable](https://github.com/ANTsX/ANTs/pull/597)