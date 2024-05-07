# ANTs NIFTI-1 I/O

ANTs uses ITK image I/O, which uses a common framework to read many different image formats. ITK has changed its approach to NIFTI headers over time. Common to all versions is that ITK does not support a full affine transform between voxel and physical coordinates. The transformation must be represented by a rotation and translation, with scaling defined by the pixel dimensions. 

The variation in NIFTI I/O comes from changes in how ITK looks for the rotation and translation information in the NIFTI header, and how it writes that information to output NIFTI files.

The main difference between ITK used in ANTs >= 2.3.5 and in 2.3.4 is that ITK now prefers the sform matrix to read and write the header transform. Previously, it used the qform header fields, but this suffers from precision issues in addition to being difficult to read. 


## NIFTI spatial-temporal units

ITK had issues with reading images that don't have mm for spatial units and seconds for temporal units. These have now been fixed but the fix has not made its way into ANTs yet. We recommend pre-processing data such that its units are mm and s. ITK also used to write out data with mm and s units, but it set the xyzt_units field incorrectly. This has also been fixed and will be included in a future ANTs release.


## Summary of NIFTI I/O for ANTs 2.3.5 and later

On read, a rotation and translation are extracted from the sform matrix if possible. Non-rigid components of the sform are not used and will not be preserved in output. The qform will be used on read if the sform is not present. 

If both sform and qform are present, sform is used if the sform_code is `NIFTI_XFORM_SCANNER_ANAT`. If the sform_code is something else, the qform and sform transform will be compared. If they are identical (within a small tolerance), then sform is still used. If the sform and qform are different, it is assumed that the sform is some other transform (eg, to MNI) and the qform represents the transform to scanner anatomical coordinates.

On write, the rotation, voxel scaling and translation for the output image is written back into the header sform and qform, and both qform and sform codes are set to NIFTI_XFORM_SCANNER_ANAT.


## How NIFTI-1 transforms are read

In all cases, the image spacing is read from the pixdim elements of the header. The rotation and translation information can either come from the qform (the quatern_[b,c,d] and qoffset_[x,y,z] NIFTI fields) or the sform matrix (srow_[x,y,z] fields).

The code for computing the transform to physical space is here: [itkNiftiImageIO.cxx](https://github.com/InsightSoftwareConsortium/ITK/blob/ceac959c2dbcb52c478c05535eba9c7ff83b5dca/Modules/IO/NIFTI/src/itkNiftiImageIO.cxx#L1783). From reviewing the code, this is my take on how the algorithm proceeds (in most use cases\*):

1. Check the sform matrix to see if it can be decomposed into a rotation matrix with scaling matching the image spacing. If a rotation matrix can be extracted, proceed to step 2. Otherwise use qform.

2. If the sform_code is NIFTI_XFORM_SCANNER_ANAT, use the sform. Otherwise, proceed to step 3.

3. If qform is NIFTI_XFORM_UNKNOWN, use sform. Otherwise proceed to step 4.

4. Check if the qform and sform transforms are very similar. If so, use sform, otherwise use qform. The idea here is to take advantage of sform's extra precision, if they are representing the same transform. Otherwise, it is assumed that the qform represents the desired transform, and the sform (with code something other than NIFTI_XFORM_SCANNER_ANAT) represents alignment to some other space. 

\* Assuming here that a valid transform can be found in at least one of the sform or qform. If no useable transform can be found, an exception is thrown. Technically, it is possible to have sform_code == NIFTI_XFORM_UNKNOWN and still use sform, if the sform is very similar to the qform (see [here](https://github.com/InsightSoftwareConsortium/ITK/blob/ceac959c2dbcb52c478c05535eba9c7ff83b5dca/Modules/IO/NIFTI/src/itkNiftiImageIO.cxx#L1876-L1879)). But this should not happen in practice because the sform transform should be zero if sform_code == NIFTI_XFORM_UNKNOWN. Lastly, see the ANALYZE legacy use case below. 


## ANALYZE backwards compatibility

In rare cases, the qform and sform code may both be NIFTI_XFORM_UNKNOWN. This is only for reading legacy ANALYZE files. In this case, neither sform nor qform is used, and the ANALYZE orientation codes will be used to define the rotation. The translation is set to zero. This is not recommended and NIFTI images should be used wherever possible. Take extra care to check correctness of processing and output when handling ANALYZE images.


## How NIFTI-1 transforms are written

Both the sform and qform parts of the NIFTI header are populated with the rotation and translation of the output space, and both the qform and sform codes are set to NIFTI_XFORM_SCANNER_ANAT. The spacing is encoded in the pixdim and the srow matrix. Transforms to other spaces (eg, MNI space) are not preserved.


## itk::ERROR: ITK only supports orthonormal direction cosines. No orthonormal definition found!

If this error arises with a seemingly valid sform, it may be because of a failed numerical check in the ITK NIFTI code (see issue #1213). There have been various efforts to prevent these in ITK, (for more detail see PRs [here](https://github.com/InsightSoftwareConsortium/ITK/pull/2701) and [here](https://github.com/InsightSoftwareConsortium/ITK/pull/3339)), but they can still happen in rare cases.

With ANTs 2.5.0 or later, advanced users may set

```
export ITK_NIFTI_SFORM_PERMISSIVE=1
```

to enable ITK to correct the sform upon read. A warning will be generated in this case, and the sform transform will be adjusted as needed.