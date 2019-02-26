# itk::ERROR: Inputs do not occupy the same physical space!

This error will occur if you attempt to use images in different physical space in a context that requires a common physical space. For example, a brain mask passed to `N4BiasFieldCorrection` should occupy the same physical space as the image being corrected, or you will see an error like this

```
Exception caught:
itk::ExceptionObject (0x2c1a7c0)
Location: "unknown"
File: /data1/lib/build/ANTS/CentOS7/antsbin/ITKv5-install/include/ITK-5.0/itkImageToImageFilter.hxx
Line: 240
Description: itk::ERROR: N4BiasFieldCorrectionImageFilter(0x2c044e0): Inputs do not occupy the same physical space!
InputImage Origin: [-9.5968330e+01, 7.5178741e+01, -4.6519180e+01], InputImage_1 Origin: [-9.5723050e+01, 7.0265378e+01, -4.5940466e+01]
Tolerance: 8.0000000e-06
InputImage Direction: 1.0000000e+00 0.0000000e+00 0.0000000e+00
0.0000000e+00 1.0000000e+00 0.0000000e+00
0.0000000e+00 0.0000000e+00 1.0000000e+00
, InputImage_1 Direction: 9.9634532e-01 8.5416688e-02 -1.9561758e-06
8.2807286e-02 -9.6590231e-01 2.4530730e-01
-2.0951447e-02 2.4441094e-01 9.6944537e-01
```

Other programs can produce similar errors if the headers do not agree.

## Causes and solutions

The obvious cause of this error is when the images are truly in different physical spaces. For example, if one image is in a subject's native T1 space, and another is in a group template space. If the T1 has been registered to the template, you can use `antsApplyTransforms` to resample the template space images into the native space, and vice versa. 

### Images are aligned but have different number of dimensions

[Dimension mismatch between images can cause direction matrices to be wrong](https://github.com/ANTsX/ANTs/issues/250)herehttps://github.com/ANTsX/ANTs/issues/250

This often happens when a multi-component image like a time series or vector image is used when a 3D scalar image is expected. Usually you will want to use a 3D image (for example, the average BOLD image or the average b=0 image from DWI) to do things like motion correction to the T1 image space.


### Precision errors 

ITK reads and writes a variety of file formats. Images are read into a generic ITK image object, and after processing the results are written out in the desired format. As a result, there can be precision errors in the output NIFTI header.

```
  ThresholdImage 3 brain.nii.gz brainMask.nii.gz 1 Inf
```

will write `brainMask.nii.gz`, but its header is created from the ITK image in memory. One could (though probably shouldn't) replace ".nii.gz" in the output with ".mha", ".nrrd", or one of the other supported formats. A diff of the headers may show small differences in the qform between `brain.nii.gz` and `brainMask.nii.gz`. This can cause errors in downstream processing, for example: 

```
Description: itk::ERROR: ImageToImageFilter(0x352ff90): Inputs do not occupy the same physical space!
InputImage Origin: [9.5322963e+01, -1.1251340e+02, -1.3474442e+02], InputImage_1 Origin: [9.5322960e+01, -1.1251340e+02, -1.3474442e+02]
Tolerance: 1.8750000e-06
```

You can use `CopyImageHeaderInformation` after processing to set headers consistently. You can also minimize problems by using a consistent reference space when warping images.