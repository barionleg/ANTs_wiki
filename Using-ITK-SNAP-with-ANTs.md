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
