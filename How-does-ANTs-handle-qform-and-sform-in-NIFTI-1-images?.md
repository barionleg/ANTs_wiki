## Short answer

ANTs uses ITK image I/O, which uses a common framework to read many different image formats. ITK supports a rotation, but not a full affine transform, to translate coordinates from index space (voxel indices) to physical space. Therefore it can only use qform when reading a NIFTI-1 image. When writing NIFTI output, the sform is overwritten with values that match the qform.


## Long answer

The exact procedure for defining the transform to physical space is here: [itkNiftiImageIO.cxx](https://github.com/InsightSoftwareConsortium/ITK/blob/285d589717dcb5252e68b90241b9c4853aa6319b/Modules/IO/NIFTI/src/itkNiftiImageIO.cxx#L1584)

In general, qform is used on input. The rotation comes from the quaternion, the scaling comes from the pixel dimensions, and the translation from the qoffset fields. This is NIFTI "method 2" as explained in the [NIFTI-1 documentation](http://nifti.nimh.nih.gov/nifti-1/documentation/nifti1fields/nifti1fields_pages/qsform.html). This is the procedure whenever the qform code is greater than 0, regardless of the sform code.

In rare cases, the qform code is 0. This should never be the case except for backwards compatibility, ie reading old ANALYZE files. If the qform code is 0, indicating no valid qform, the rotation and scaling are extracted from the sform, if it is present.

When writing NIFTI files, ITK encodes the rotation, translation and scaling in both the qform and sform ([see here for the code](https://github.com/InsightSoftwareConsortium/ITK/blob/285d589717dcb5252e68b90241b9c4853aa6319b/Modules/IO/NIFTI/src/itkNiftiImageIO.cxx#L1705)). The qform code is set to 2, and the sform code is set to 1. The loss of the sform is unavoidable because the ITK I/O code reads images into a common internal structure, independent of the individual image format on disk. This allows ITK to support many file formats transparently, but it limits the information that can be carried through to the output.

## Potential interoperability problems with other software

1. qform and sform codes may be interpreted differently. ITK uses qform whenever qform_code > 0.

2. The sform will be overwritten in output images, and sform code will be set to 1. If the sform is not identical to the qform, information will be lost. 

3. Another [issue](https://github.com/ANTsX/ANTs/issues/500):  "Note that in cases where both an SForm and QForm are present, ITK gives precedence to the QForm while SPM/MRIcroGL/FSLeyes/Mango give precedence to the SForm. The rationale for the SPM approach is clear: QForms were designed to provide an idea of native space, while the SForm could provide the user an easy way to provide a starting estimate. When informed of this difference the ITK group elected to keep their method to ensure backward compatibility. To support both conventions, recent versions of SPM change both the SForm as well as QForm when the user changes the origin with the "Display" button. While the original intent of the SForm and QForm was to provide a method to encode both native scanner as well as some losslessly re-oriented coordinates, the differences in implementations supports SPM's method of using the same value for both to ensure consistent display and processing."