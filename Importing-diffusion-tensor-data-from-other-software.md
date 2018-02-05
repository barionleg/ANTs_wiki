## NIFTI and other file formats

The discussion on this page is mostly about the NIFTI-1 file format. Tensors in other file formats supported by ITK (such as MetaIO or NRRD) may also work, but are less likely to have been tested. If you have used other formats successfully and have example code to include below, please let us know.


## NIFTI DT image

The NIFTI file should be 5D and have dimensions [X,Y,Z,1,6] where X,Y,Z are the number of voxels. The intent code should be set to NIFTI_INTENT_SYMMATRIX. The NIFTI format specifies a symmetric matrix in lower triangular format. For the DT, this means the six copmonents of the image are ordered [dxx, dxy, dyy, dxz, dyz, dzz]. 


## Image and tensor coordinate system

The NIFTI header describes the transformation of the image voxels into physical space, as for scalar images. The coordinate system of the tensors may be different. This can result in tensors that have the correct location in the image, but incorrect orientation. This is most notable in the white matter, where the tensor principal eigenvectors should be aligned with the major white matter bundles.

The correct convention for the tensor coordinate system is software dependent and problems usually result from the DT reconstruction reading gradient vectors (`bvecs` in FSL terminology) that are not correctly formatted. See [here](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FDT/FAQ#What_conventions_do_the_bvecs_use.3F) and [here](https://users.fmrib.ox.ac.uk/~paulmc/fsleyes/userdoc/latest/troubleshooting.html#line-vectors-tensors-fibre-orientation-distributions-are-left-right-flipped) for discussion of this issue in FSL and [here](http://camino.cs.ucl.ac.uk/index.php?n=Tutorials.DTI) for an example using Camino.

The ANTs program `RebaseTensorImage` can convert tensors that are stored in voxel space to physical space, and vice versa.


## Examples

Below are some examples of how to fit diffusion tensors in the correct NIFTI format for ANTs, using some common diffusion software packages.


### FSL

```
dtifit -k dwi.nii.gz -o dti -m mask.nii.gz -r bvecs -b bvals --save_tensor
```

This produces a tensor component image `dti_tensor.nii.gz` which is stored as a **4D** NIFTI image in upper-triangular order. This needs to be converted into a 5D lower-triangular image for ANTs.