## __Problem:__  multiple runs of ``antsRegistration`` on the same images produces different results

## Variance due to floating point precision errors

In all scientific computing, the limited precision of floating point operations introduces variance that can be mitigated (often at the cost of performance), but usually not eliminated. Strategies such as [compensated summation](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) exist to manage precision errors.  

Floating point precision may cause differences in the registration solution on the same images in several contexts including

* Multi-threading. The exact sequence of floating point operations depends on the number of threads, as the results of computations performed in parallel are combined.

* User choice of single or double precision for floating point operations.

* Differences in CPU architecture.

* Differences in compiler options for ANTs, ITK, or underlying system libraries.


## Variance due to random sampling

Random sampling is used in various contexts, and different runs will produce a different sequence of random numbers, unless a fixed seed is used. In registration, the point set used to evaluate similarity metrics is initialized as the grid of voxel centers of the fixed image. The point set is then perturbed randomly to [reduce bias in estimation](http://bigwww.epfl.ch/preprints/thevenaz0602p.pdf).


## Strategies to improve reproducibility

* Set a fixed seed for randomization in [ITK](https://github.com/InsightSoftwareConsortium/ITK/blob/8a2a15f41218c925c0a89119e09419d48f83eb22/Modules/Registration/RegistrationMethodsv4/include/itkImageRegistrationMethodv4.hxx#L940-L949).

* Use double precision.

* Disable multi-threading.

* Use a consistent computer infrastructure (same hardware, OS, compilation of ANTs etc).

However, each of these has it's own drawbacks:

* How to choose the seed since each fixed seed is just as good as the next one.

* Increased storage of fixed/moving images and warp fields (forward/inverse displacement fields to/from (middle) virtual domain--total of 4 displacement fields in storage during optimization).

* Computation time is increased by restricting threads or memory requirements.

* Dense sampling or a fixed seed may trade variance for bias.


## Quantification of variance in registration results

The sources of variance in a simple registration task appear to be (in decreasing order of magnitude):

 * Random point set sampling / perturbation. This can be removed by dense sampling or use of a fixed seed.

 * Single vs double precision

 * Multi-threading

More details will appear here. 


## Related discussion threads

* [antsRegistration does not produce equivalent results](https://github.com/ANTsX/ANTsR/issues/210#issuecomment-377511054)

* [Non-deterministic result of antsAffineInitializer](https://github.com/ANTsX/ANTs/issues/444)

* [Fix registration ANTs random seed with an environment variable](https://github.com/ANTsX/ANTs/pull/597)

