ANTs lacks dcm to dwi to dti functionality.  We previously relied on camino but this software is no longer maintained, afaik.   thus [mrtrix](http://mrtrix.readthedocs.io/).

brief notes on how to go from dicom to nifti files we can use in ants.

useful programs

```
mrconvert sid/ep2d_64dir/date/ptid /tmp/my.mif                                # convert dcm to mif

dwidenoise /tmp/my.mif /tmp/myd.mif                                           # mysterious denoising method for dwi

dwiextract /tmp/myd.mif - -bzero | mrmath - mean /tmp/mymeanb0.nii.gz -axis 3 # mean b0 image

dwipreproc -rpe_none /tmp/my.mif /tmp/myp.mif                                 # preprocess dwi e.g. with motion (needs fsl)

dwi2mask /tmp/myd.mif /tmp/mymask.nii.gz                                      # mask dwi (not reliably)

dwi2tensor /tmp/myd.mif /tmp/mydti.nii.gz                                     # convert to dti

tensor2metric -fa /tmp/myfa.nii.gz /tmp/mydti.nii.gz                          # get fa from dti
```
one might also do [warping with ANTs or other programs](http://mrtrix.readthedocs.io/en/latest/spatial_normalisation/warping_images_with_warps_from_other_packages.html)

i would like to know how to apply motion correction ...