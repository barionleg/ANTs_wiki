## Obtaining ITK-SNAP 

[ITK-SNAP](http://itksnap.org) is free software for image segmentation, which is also a very useful platform for viewing input to ANTs and the output of ANTs processing. Documentation and tutorials on the uses of ITK-SNAP are provided on the SNAP website. Here we show some basic operations for checking image alignment.

ITK-SNAP uses ITK I/O, like ANTs. It is therefore a good tool for previewing how ANTs will interpret a particular image. 


## Previewing image orientation 

Loading the image into SNAP, you can see how ITK interpreted the physical space of the image. The actual orientation of the image on the screen is a display convention. For example, anatomical right is screen left. What matters is that the labels (L, R, A, P, S, I) agree with the anatomical alignment. If these are correct, then ANTs should have no problems with the image.

If the anatomical labels are not correct, most ANTs processing might still work, as long as all the data is internally consistent. But it's safer to get the orientation correct from the outset. This is particularly important when dealing with data where other geometric information (like gradient directions for diffusion) need to be interpreted correctly.


## Previewing images before registration

To preview the initial alignment of two images, load the fixed and moving images in two separate SNAP windows. It is not sufficient to load the images as overlays in a single SNAP window, because in that case SNAP will attempt to re-use header information from the main image, and this can hide problems.

By default, the two windows will be linked, meaning SNAP will attempt to align the crosshairs position in physical space between the two images. If you click a point in one image, the cursor should move in both. Verify that the images have the same orientation, and that there is overlap between the foreground of the images.

If the orientation is not consistent (for example, the image is flipped left-right or anterior-posterior), this needs to be corrected. See `PermuteFlipImageOrientationAxes` and `CopyImageHeaderInformation`. 

If the orientation is consistent but the crosshairs indicate little or no overlap, `antsRegistration` _may_ be able to compensate by aligning the images using the `-r` option. This can perform an initial translation based on the geometric center, or the center of mass, of the images. This can be evaluated by running a registration with zero iterations:

```
antsRegistration -d 3 -r [ fixed.nii.gz , moving.nii.gz , 1] -t Rigid[0.1] \
-m MI[ fixed.nii.gz, moving.nii.gz, 1, 32] -c 0 -f 4 -s 2 \
-o [movingInit, movingInit_deformed.nii.gz] 
```

Here the option to `-r` uses the center of mass to align the images.


## Viewing registration output

After registration, the deformed image produced by `antsRegistration` or `antsApplyTransforms` (with the forward warps) will be resampled into the fixed image space. The deformed and fixed image will therefore always have the same header. They can be loaded into a single SNAP session with one image as the main image. The images can be viewed side by side or with one as an overlay. The overlay can be adjusted in opacity and toggled hidden / visible (see [ITK-SNAP keyboard shortcuts](http://www.itksnap.org/pmwiki/uploads/Documentation/snap_shortcuts_v3.pdf)).


## Visualizing warp fields

Warp fields may be loaded as overlays and viewed as vector images (displayed as color-coded vectors, magnitude, or component images) or as a deformation grid. To access these options, right-click the warp field image in the "Cursor Inspector", or navigate to the layer inspector from the Tools menu.


## Physical space coordinates

The cursor position in the toolbox on the left side of the screen is voxel coordinates. Physical space coordinates are shown in the image inspector, available from the Tools menu under "Image Information". 

Point set operations in ANTs require points in ITK physical space, which are listed in SNAP as "World units (ITK)".


## Visualizing diffusion tensors

To compute and visualize diffusion tensor statistics, something like

```
ImageMath dtFA.nii.gz TensorFA dt.nii.gz
ImageMath dtMD.nii.gz TensorMeanDiffusion dt.nii.gz
ImageMath dtRGB.nii.gz TensorColor dt.nii.gz
itksnap -g dtFA.nii.gz -o dtMD.nii.gz dtRGB.nii.gz
```

To display the RGB map as color, go to the Layer Inspector and then click on the RGB overlay, and change display mode to RGB.

An important limitation of this approach is that flips in the coordinate system of the tensors cannot be detected, because the RGB mapping is insensitive to the sign of the vector components. However, you will be able to tell if the tensor components are incorrectly ordered, or if the brain has been flipped.

### Loading the tensor image directly

ITK-SNAP can load the diffusion tensor directly, as a multiple component image. When it does this, it will display the components in **upper-triangular** order, which is what ITK uses internally. This can be confusing because the NIfTI images on disk are required to be in lower-triangular order. So if you are loading a NIfTI tensor into ITK-SNAP, it should display in upper-triangular order.