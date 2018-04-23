ANTs has been developed over many years and several programs and scripts are no longer actively maintained. Generally, ANTs policy is to leave these programs available for backwards compatibility and reproducibility.

In the list below, the currently maintained version of the program or script is listed in parentheses, where available. 

#  Executables

## ANTS (antsRegistration)

`ANTS` was the registration command in the original release of ANTs. It is no longer maintained, but is still used in the scripts `antsMultivariateTemplateConstruction.sh`. 

With ITK version 4, a variety of new registration filters were added. The interface to these "v4" algorithms is `antsRegistration`. This program allows the user much more flexibility to alter parameters, and is used in the current ANTs scripts. However, it is more computationally expensive to run, so users with large images or time constraints might still find ANTS useful.

See also: 

 * `WarpImageMultiTransform`, used with ANTS to deform images. `antsApplyTransforms` is forward compatible.
 *  [ANTS and antsRegistration](https://github.com/ANTsX/ANTs/wiki/ANTS-and-antsRegistration) for information on some important parameter choices between the two scripts.


## ANTSJacobian (CreateJacobianDeterminantImage)

`CreateJacobianDeterminantImage` is the older program it is still used in the ANTs scripts. `ANTSJacobian` was intended to replace it but is no longer under active development.


## KellySlater (KellyKapowski) 

Old cortical thickness implementation, named after a surfer. Replaced by a version named after a TV character. Both are implementations of [DiReCT](https://www.ncbi.nlm.nih.gov/pubmed/19150502).


## N3BiasFieldCorrection (N4BiasFieldCorrection)

Bias correction based on the famous N3 algorithm by Sled et al. See the [paper](https://www.ncbi.nlm.nih.gov/pubmed/20378467) for details of N4 and a comparison to N3 performance.

   
## WarpImageMultiTransform, WarpTensorImageMultiTransform, WarpTimeSeriesImageMultiTransform, ComposeMultiTransform  (antsApplyTransforms)

These have been replaced by `antsApplyTransforms`, which can perform multiple functions. Note that tensor images still need to be reoriented after warping, using `ReorientTensorImage`.


# Scripts

## buildtemplateparallel.sh (antsMultivariateTemplateConstruction.sh)

The new implementation `antsMultivariateTemplateConstruction.sh` is fairly similar but supports multi-parametric templates, eg T1 and T2 images from the same set of subjects can be used to build a T1 and T2 template simultaneously.

The newer `antsMultivariateTemplateConstruction2.sh` uses `antsRegistration` in place of ANTs.

# Programs / scripts no longer included in ANTs

These have been removed from the default build but may exist in old code or be referenced in papers

## antsMalfLabeling.sh (antsJointLabelFusion.sh)

Script for multi-atlas segmentation.

## jointfusion (antsJointFusion)

Executable called in deprecated `antsMalfLabeling.sh`. 