## NIFTI and other file formats

The discussion on this page is mostly about the NIFTI-1 file format. Tensors in other file formats supported by ITK (such as MetaIO or NRRD) may also work, but are less likely to have been tested. If you have used other formats successfully and have example code to include below, please let us know.


## NIFTI DT image

The NIFTI file should be 5D and have dimensions [X,Y,Z,1,6] where X,Y,Z are the number of voxels. The intent code should be set to NIFTI_INTENT_SYMMATRIX. The NIFTI format specifies a symmetric matrix in lower triangular format. For the DT, this means the six components of the image are ordered [dxx, dxy, dyy, dxz, dyz, dzz]. 


## Image and tensor coordinate system

The header describes the transformation of the image voxels into physical space, as for scalar images. In ITK, this is called the "Direction" matrix, and is always a rigid transform (affine transforms, like those defined in the NIFTI "sform", are not supported). 

The correct convention for the tensor coordinate system is software dependent, and extracting them from the raw scaner data has required considerable effort from the creators of tools like `dcm2niix`. See [here](https://www.nitrc.org/plugins/mwiki/index.php/dcm2nii:MainPage#Diffusion_Tensor_Imaging) for discussion and sample data.

On the post-processing side, see [here](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FDT/FAQ#What_conventions_do_the_bvecs_use.3F) and [here](https://users.fmrib.ox.ac.uk/~paulmc/fsleyes/userdoc/latest/troubleshooting.html#line-vectors-tensors-fibre-orientation-distributions-are-left-right-flipped) for bvec requirements in FSL and [here](http://camino.cs.ucl.ac.uk/index.php?n=Tutorials.DTI) for an example using Camino. 

Errors in the bvecs can go unnoticed because invariant scalar characteristics like fractional anisotropy and mean diffusion are not affected, but tractography will be affected. 


# Converting tensors to ANTs format

The examples below describe how to convert the tensors into the correct file format for ANTs. 

To work with ANTs, the tensors should be in index space, such that a tensor can be correctly reoriented into physical space by applying the direction matrix. 


## FSL

```
dtifit -k dwi.nii.gz -o dti -m mask.nii.gz -r bvecs -b bvals --save_tensor
```

This produces a tensor component image `dti_tensor.nii.gz` which is stored as a **4D** NIFTI image in upper-triangular order. This needs to be converted into a 5D lower-triangular image for ANTs by first splitting the 4D image into 3D components, and then labeling them by their matrix indices [xx,xy,xz,yy,yz,zz]. Then `ImageMath` can combine them into a NIFTI_SYMMATRIX image.

```
ImageMath 4 dtiComp.nii.gz TimeSeriesDisassemble dti_tensor.nii.gz
i=0 
for index in xx xy xz yy yz zz; do
   mv dtiComp100${i}.nii.gz dtiComp_${index}.nii.gz
   i=$((i+1))
done
ImageMath 3 dtAnts.nii.gz ComponentTo3DTensor dtiComp_
```

As stated [here](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FDT/FAQ#What_conventions_do_the_bvecs_use.3F), FSL always requires a radiological (left-handed) coordinate system for bvecs, regardless of whether the DWI image is radiological or neurological. You can check the orientation of the data with:

```
fslorient dwi.nii.gz
```

If this reports `RADIOLOGICAL`, and output of `dtifit` looks correct in `fsleyes`, then the tensors should be compatible with ANTs. If `fslorient` reports `NEUROLOGICAL`, the bvecs still have to be radiological for FSL, but then image and bvec coordinate system will not agree and the tensors will not be correctly oriented by ANTs. 


## Mrtrix

The procedure is similar to FSL but the [https://mrtrix.readthedocs.io/en/latest/reference/commands/dwi2tensor.html](dwi2tensor) command uses a different component ordering, so the iteration over components is different:

```
ImageMath 4 dtiComp.nii.gz TimeSeriesDisassemble dt.nii.gz
i=0 
for index in xx, yy, zz, xy, xz, yz; do
   mv dtiComp100${i}.nii.gz dtiComp_${index}.nii.gz
   i=$((i+1))
done
ImageMath 3 dtAnts.nii.gz ComponentTo3DTensor dtiComp_
```


## Camino

```
modelfit -model ldt_wtd -inputfile dwi.nii.gz -schemefile a.scheme \
  -brainmask mask.nii.gz -outputfile wdt.Bdouble
dt2nii -inputfile wdt.Bdouble -header dwi.nii.gz -outputroot nifti_
```

This will produce nifti_dt.nii.gz, which is in the required format. The bvecs should agree with the voxel ordering of the image. Camino uses the voxel space internally and uses the NIFTI header to convert streamline tracts into physical space.