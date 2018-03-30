### __Problem:__  multiple runs of ``antsRegistration`` on the same images produces different results

Some of the relevant issues:

* Floating point precision and multi-threading addressed via [compensated summation](https://en.wikipedia.org/wiki/Kahan_summation_algorithm).  

* Random perturbations from sampling on the grid [improve results and mitigate an estimation bias problem](http://bigwww.epfl.ch/preprints/thevenaz0602p.pdf).


One could improve reproducibility results by:

* setting a fixed seed for randomization in [ITK](https://github.com/InsightSoftwareConsortium/ITK/blob/8a2a15f41218c925c0a89119e09419d48f83eb22/Modules/Registration/RegistrationMethodsv4/include/itkImageRegistrationMethodv4.hxx#L940-L949)

* single threading.

* use double precision.

However, each of these has it's own drawbacks:

* How to choose the seed since each fixed seed is just as good as the next one.

* computation time is increased over multi-threading.

* increased storage of fixed/moving images and warp fields (forward/inverse displacement fields to/from (middle) virtual domain--total of 4 displacement fields in storage during optimization).

Relevant threads:

* [antsRegistration does not produce equivalent results](https://github.com/ANTsX/ANTsR/issues/210#issuecomment-377511054)

* [Non-deterministic result of antsAffineInitializer](https://github.com/ANTsX/ANTs/issues/444)
