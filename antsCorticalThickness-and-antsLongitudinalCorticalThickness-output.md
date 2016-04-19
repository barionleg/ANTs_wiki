# antsCorticalThickness.sh

`antsCorticalThickness.sh` output is to a single directory, with multiple files.

Definitions

**Subject space**
**Template space**

Some files are produced only if optional input parameters are set. The full file name will be `${outputPrefix}description${outputSuffix}`, the prefix and suffix are set at run time by the user.

* `BrainExtractionMask`- Brain extraction mask in subject space.

* `BrainNormalizedToTemplate` - Extracted brain image normalized to the template space. 

* `BrainSegmentation0N4` - Input to the segmentation algorithm. It is not brain extracted, but is bias-corrected. If multiple images are used for segmentation, there will be `BrainSegmentation1N4` and so on. The brain extracted version of this is `ExtractedBrain0N4`.

* `BrainSegmentation` - Segmentation image, one label per tissue class. The number of classes is determined by the input priors.

* `BrainSegmentationPosteriors1` - Posterior probability of class 1 (usually CSF). A similar image is produced for all classes. The numbering scheme matches the input priors.

* `CorticalThickness` - Cortical thickness image in subject space.

* `CorticalThicknessNormalizedToTemplate` - Cortical thickness image in template space.

* `ExtractedBrain0N4` - Brain-extracted version of `BrainSegmentation0N4`.

* `SubjectToTemplate1Warp`, `SubjectToTemplate0GenericAffine.mat` - Transforms to be used when warping images from the subject space to the template space (see below).

* `SubjectToTemplateLogJacobian` - Log of the determinant of the Jacobian, quantifies volume changes in the subject to template warp.

* `TemplateToSubject0Warp`, `TemplateToSubject0GenericAffine.mat` - Transforms to be used when warping images from the template to the subject space (see below).


## Applying the warps

To warp an image from the subject to template space:

To warp an image from the template to subject space: