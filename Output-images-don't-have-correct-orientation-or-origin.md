Some potential causes of problems with output images being flipped or translated.

### NIfTI header codes not set appropriately for ANTs

ITK, and therefore ANTs, only supports a rigid transform to physical space. It prefers qform over sform. See

https://github.com/stnava/ANTs/wiki/How-does-ANTs-handle-qform-and-sform-in-NIFTI-1-images%3F


### Dimension mismatch

If you pass a 4D image where a 3D image is expected, the transform of the output image may not be correct. See

https://github.com/stnava/ANTs/issues/250

Better to split the 4D volume into components and process them independently (can be done with `ImageMath`).