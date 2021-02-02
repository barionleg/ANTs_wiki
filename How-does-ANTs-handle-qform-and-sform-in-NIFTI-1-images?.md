# ANTs NIFTI-1 I/O

ANTs uses ITK image I/O, which uses a common framework to read many different image formats. ITK has changed its approach to NIFTI headers over time. Common to all versions is that ITK does not support a full affine transform between voxel and physical coordinates. The transformation must be represented by a rotation and translation, with scaling defined by the pixel dimensions. 

The variation in NIFTI I/O comes from changes in how ITK looks for the rotation and translation information in the NIFTI header, and how it writes that information to output NIFTI files.

The main difference between ITK used in ANTs 2.3.5 and in 2.3.4 is that ITK now prefers the sform matrix to read and write the header transform. Previously, it used the qform header fields, but this suffers from precision issues in addition to being difficult to read. 

## Summary of NIFTI I/O for ANTs 2.3.5 and later

On read, a rotation and translation are extracted from the sform matrix if possible. Non-rigid components of the sform are not used and will not be preserved in output. The qform will be used on read if the sform is not present or has a code indicating it is a transform to some other space.

On write, the rotation and translation for the output space is written back into the sform and qform, and both qform and sform codes are set to NIFTI_XFORM_SCANNER_ANAT.  


## How NIFTI-1 transforms are read

In all cases, the image spacing is read from the pixdim elements of the header. The rotation and translation information can either come from the qform (the quatern_[b,c,d] and qoffset_[x,y,z] NIFTI fields) or the sform matrix (srow_[x,y,z] fields).

The exact procedure for computing the transform to physical space is here: [itkNiftiImageIO.cxx](https://github.com/InsightSoftwareConsortium/ITK/blob/master/Modules/IO/NIFTI/src/itkNiftiImageIO.cxx). From reviewing the code, this is my take on how the algorithm proceeds:

1. Check the sform matrix to see if it can be decomposed into a rotation matrix with scaling matching the image spacing. If so, the rotation is extracted. If not, use the qform transform.

2. If the sform_code is NIFTI_XFORM_SCANNER_ANAT, use the sform rotation + translation.

3. If both the sform_code and qform_code are not NIFTI_XFORM_UNKNOWN, check if the qform and sform transforms are very similar. If so, use sform, otherwise use qform. The idea here is to take advantage of sform's extra precision, if they are representing the same transform. Otherwise, it is assumed that the qform represents the desired transform, and the sform (with code something other than NIFTI_XFORM_SCANNER_ANAT) represents alignment to some other space.

In rare cases, the qform and sform code may both be NIFTI_XFORM_UNKNOWN. This is only for reading legacy ANALYZE files. In this case, the old Analyze orientation codes will be used to define the rotation. The translation is set to zero. This is not recommended and extra care should be taken to check the results.


## How NIFTI-1 transforms are written

Both the sform and qform parts of the NIFTI header are populated with the rotation and translation of the output space, and both the qform and sform codes are set to NIFTI_XFORM_SCANNER_ANAT. The spacing is encoded in the pixdim and the srow matrix. Transforms to other spaces (eg, MNI space) are not preserved.


---

The text below describes the previous ITK behavior, which was similar but preferred to read the qform if it was present. The text below is valid for ANTs 2.3.x. On output, the transform was written to the qform, and the sform was set to zeros.  

Before ANTs 2.3.0, qform was still used on input, but the sform was set in the output to be identical to the qform.


# ITK NIFTI I/O for ANTs 2.3.[0-4] 



## Short answer

ANTs uses ITK image I/O, which uses a common framework to read many different image formats. ITK supports a rotation, but not a full affine transform, to translate coordinates from index space (voxel indices) to physical space. Therefore it can only use qform when reading a NIFTI-1 image. When writing NIFTI output, only the qform is written; the sform is replaced with zeros.


## Long answer

The exact procedure for defining the transform to physical space is here: [itkNiftiImageIO.cxx](https://github.com/InsightSoftwareConsortium/ITK/blob/master/Modules/IO/NIFTI/src/itkNiftiImageIO.cxx)

In general, qform is used on input. The rotation comes from the quaternion, the scaling comes from the pixel dimensions, and the translation from the qoffset fields. This is NIFTI "method 2" as explained in the [NIFTI-1 documentation](http://nifti.nimh.nih.gov/nifti-1/documentation/nifti1fields/nifti1fields_pages/qsform.html). This is the procedure whenever the qform code is greater than 0, regardless of the sform code.

In rare cases, the qform code is 0. This should never be the case except for backwards compatibility, ie reading old ANALYZE files. If the qform code is 0, indicating no valid qform, the rotation and scaling are extracted from the sform, if it is present.

When writing NIFTI files, ITK encodes the rotation, translation and scaling in the qform. The qform code is set to 1 (NIFTI_XFORM_SCANNER_ANAT), and the sform code is set to 0. The loss of the sform is unavoidable because the ITK I/O code reads images into a common internal structure, independent of the individual image format on disk, and only supports a rigid transformation.

## Precision errors in header transforms

ITK reads and writes a variety of file formats. Images are read into a generic ITK image object, and after processing the results are written out in the desired format. As a result, [there can be precision errors in the output NIFTI header](https://github.com/ANTsX/ANTs/wiki/Inputs-do-not-occupy-the-same-physical-space). 


## Potential interoperability problems with other software

1. qform and sform codes may be interpreted differently. ITK uses qform whenever qform_code > 0.

2. The sform will be overwritten in output images, and sform code will be set to 0. If the sform is not identical to the qform, information will be lost. 

3. Another [issue](https://github.com/ANTsX/ANTs/issues/500):  "Note that in cases where both an SForm and QForm are present, ITK gives precedence to the QForm while SPM/MRIcroGL/FSLeyes/Mango give precedence to the SForm. The rationale for the SPM approach is clear: QForms were designed to provide an idea of native space, while the SForm could provide the user an easy way to provide a starting estimate. When informed of this difference the ITK group elected to keep their method to ensure backward compatibility. To support both conventions, recent versions of SPM change both the SForm as well as QForm when the user changes the origin with the "Display" button. While the original intent of the SForm and QForm was to provide a method to encode both native scanner as well as some losslessly re-oriented coordinates, the differences in implementations supports SPM's method of using the same value for both to ensure consistent display and processing."

4. In registration, the fixed image may have an sform, which will be ignored in the registration. The warped moving image will have only a qform. Thus the registration may work correctly but appear to be incorrect because of differences in how the NIFTI header is interpreted.