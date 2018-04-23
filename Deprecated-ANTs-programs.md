ANTs has been developed over many years and several programs and scripts are no longer actively maintained. Generally, ANTs policy is to leave these programs available for backwards compatibility and reproducibility.

In the list below, the currently maintained version of the program or script is listed in parentheses, where available. 

#  Executables

## ANTS (antsRegistration)

`ANTS` was the registration command in the original release of ANTs. It is no longer maintained, but is still used in the scripts `antsMultivariateTemplateConstruction.sh`. 

With ITK version 4, a variety of new registration filters were added. The interface to these "v4" algorithms is `antsRegistration`. This program allows the user much more flexibility to alter parameters, and is used in the current ANTs scripts. However, it is more computationally expensive to run, so users with large images or time constraints might still find ANTS useful.

See also: 

 * `WarpImageMultiTransform`, used with ANTS to deform images. 
 *  [ANTS and antsRegistration](https://github.com/ANTsX/ANTs/wiki/ANTS-and-antsRegistration) for information on some important parameter choices between the two scripts.
   

## WarpImageMultiTransform (antsApplyTransforms)

Replaced by `antsApplyTransforms`.


# Scripts

