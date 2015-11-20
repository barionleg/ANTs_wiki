User asked how to:

"1) normalize a patient´s T1 (lets call it T1.nii) to the MNI template (lets call it MNI.nii) by obtaining a transformation/deformation matrix (lets call it T1_to_MNI)"

* `antsRegistrationSyN.sh -d 3 -f MNI.nii.gz -m T1.nii.gz -o XXX `
    * i would recommend instead 2 stages of registration: T1 to customTemplate to MNI and then concatenating these via `antsApplyTransforms`

"2) applying the T1_to_MNI matrix to T1.nii "

*  `antsApplyTransforms -d 3 -r MNI.nii.gz -i T1.nii.gz -e 0 -t XXX1Warp.nii.gz -t XXX0GenericAffine.mat ` 

"3) applying the T1_to_MNI matrix to the fMRI.nii data from the same patient (that have already been pre-registered to the respective T1.nii) "

*  `antsApplyTransforms -d 3 -r MNI.nii.gz -i fMRI.nii.gz -e 3 -t -t XXX1Warp.nii.gz -t XXX0GenericAffine.mat `   

see documentation for details, e.g. the links at the [top of this page](http://stnava.github.io/ANTs/)
