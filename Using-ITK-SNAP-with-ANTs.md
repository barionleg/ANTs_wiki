## Obtaining ITK-SNAP 

[ITK-SNAP](http://itksnap.org) is free software for image segmentation, which is also a very useful platform for viewing input to ANTs and the output of ANTs processing. Detailed documentation and tutorial on the uses of ITK-SNAP are provided on the SNAP website. Here we show some basic operations for checking image alignment.

ITK-SNAP uses ITK I/O, like ANTs. It is therefore the best tool for previewing how ANTs will interpret a particular image. 


## Previewing image orientation 

Loading the image into SNAP, you can see how ITK interpreted the physical space of the image. The actual orientation of the image is a display convention - what matters is the anatomical labels (L, R, A, P, S, I). If these are correct, then ANTs should have no problems with the image.

If the anatomical labels are not correct, most ANTs processing might still work, as long as all the data is internally consistent. But it's safer to get the orientation correct from the outset. This is particularly important when dealing with data where other geometric information (like gradient directions for diffusion) need to be interpreted correctly.


## Previewing images before registration

Load the fixed and moving images in two separate SNAP windows.


## Viewing registration output


## Evaluating the output of antsCorticalThickness 