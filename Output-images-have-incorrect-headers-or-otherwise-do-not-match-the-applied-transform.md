[ITK-SNAP](http://itksnap.org) uses similar I/O to ANTs, and is very useful for testing how ANTs will interpret the image header. See the page on [Using ITK-SNAP with ANTs](https://github.com/stnava/ANTs/wiki/Using-ITK-SNAP-with-ANTs).

Some potential causes of problems with output images being flipped or translated:

### NIfTI header codes not set appropriately for ANTs

ITK, and therefore ANTs, only supports a rigid transform to physical space. It prefers qform over sform. See

https://github.com/stnava/ANTs/wiki/How-does-ANTs-handle-qform-and-sform-in-NIFTI-1-images%3F


### Dimension mismatch

If you pass a 4D image where a 3D image is expected, the transform of the output image may not be correct. See

https://github.com/ANTsX/ANTs/wiki/Dimensionality-of-ANTs-algorithms

