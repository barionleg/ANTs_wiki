## Using ANTs templates in SPM

SPM requires templates to be in MNI space. A custom template from ANTs needs to be be affinely aligned to the MNI152 template before using the template in SPM. 

See

  https://github.com/ANTsX/ANTs/issues/590


## Other issues

Differing conventions on header transforms may become an issue for interoperability, see 

  https://github.com/ANTsX/ANTs/wiki/How-does-ANTs-handle-qform-and-sform-in-NIFTI-1-images%3F


