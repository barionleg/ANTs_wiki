# Answers to FAQs about registration

* Formulate a registration problem that ANTs can solve (at least to a good approximation). More theory [here](http://www.ncbi.nlm.nih.gov/pubmed/17659998) but basically there needs to be a sensible mapping between points in the moving and the fixed image. This requirement is violated when the same anatomy doesn't exist in both images, for example if you register a brain-extracted image to another image including the whole head. 

* Use an appropriate similarity metric. This is also part of defining a solvable problem. Mattes Mutual Information is the most general and works within or across modalities. Cross correlation (CC) can work better (especially for deformable registration) but it assumes a correlation between image intensities when the images are aligned. Thus it is often not suitable to align images from different modalities.  Several metrics are available; but Mattes and CC will cover many common use cases.

* Check the images have the same basic orientation. The exact position and orientation will vary (eg, due to head position) but they should not be upside down or back to front, and both the fixed and moving images should agree on left-right orientation. You can check this with [ITK-SNAP](http://itksnap.org), which uses the same ITK I/O as ANTs.

* Provide a good initialization for affine registration with `-r`, if that is insufficient use `antsAI`. 

* Provide a good initialization for deformable registration by running rigid and affine stages first. 

* Pad the fixed image with empty space, if necessary. If the fixed image foreground extends all the way to the boundary of the image volume, the image similarity at the edge becomes unstable because points outside the image volume are ignored, and there is a hard constraint that the deformation field is zero at the edge of the image domain. Adding extra space allows the edges of the images to be aligned better. You can also pad the moving images if necessary, but in many cases it's sufficient to have at least 10 voxels of background on all sides of the fixed image. 

* A registration mask can help with some problems such as structured background noise or anatomy of no interest, or inconsistent anatomical fields of view. Masks can also reduce registration quality as only the voxels inside the mask are considered by the registration. If possible, there should be some background voxels in the mask so that edges can be properly aligned. You may also apply masks at each stage of registration, using a wider mask (or none at all) for the rigid or affine stages, and a more restrictive mask for deformable stages.

# Debugging registration failures

The first priority is to isolate the problem. Many failures result from bad initialization or problems with the affine registration. 

*  If you've already run a deformable registration, try applying only the affine part of the transform with `antsApplyTransforms`, this will tell you if there is a problem somewhere during the affine stage. If the affine transform looks good, you can initialize the registration with that transform and experiment with the deformable part, by passing `-r myAffine0GenericAffine.mat` to `antsRegistration`.

* Initialization can be tested by running a rigid registration with zero iterations, for example:

```
antsRegistration -d 3 -r [ fixed.nii.gz , moving.nii.gz , 1] -t Rigid[0.1] \
-m MI[ fixed.nii.gz, moving.nii.gz, 1, 32] -c 0 -f 4 -s 2 \
-o [movingInit, movingInit_deformed.nii.gz] 
```
The initialization should provide a sensible translation of the moving image. 

* With good initialization, begin running the rigid registration, then the affine. Isolate where the registration converges too soon (fails to improve) or diverges (moving image gets further away from the correct solution). 