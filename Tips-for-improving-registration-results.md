# Answers to FAQs about registration

* Formulate a registration problem that ANTs can solve (at least to a good approximation). More theory [here](http://www.ncbi.nlm.nih.gov/pubmed/17659998) but basically there needs to be a sensible mapping between points in the moving and the fixed image. This requirement is violated when the same anatomy doesn't exist in both images, for example if you register a brain-extracted image to another image including the whole head.

* Provide a good initialization for affine registration with `-r`. 

* Provide a good initialization for deformable registration by running rigid and affine stages first. 

* Pad the fixed image with empty space, if necessary. This allows image similarity to be better formulated, because less of the moving image will be outside of the fixed image volume. Usually, 10 voxels of background around a head or brain template in all directions is sufficient. 


# Debugging registration failures

The first priority is to isolate the problem. Many failures result from bad initialization or problems with the affine registration. 

*  If you've already run a deformable registration, try applying only the affine part of the transform with `antsApplyTransforms`, this will tell you if there is a problem somewhere during the affine stage.

* Initialization can be tested by running a rigid registration with zero iterations, for example:

```
antsRegistration -d 3 -r [ fixed.nii.gz , moving.nii.gz , 1] -t Rigid[0.1] \
-m MI[ fixed.nii.gz, moving.nii.gz, 1, 32] -c 0 -f 4 -s 2 \
-o [movingInit, movingInit_deformed.nii.gz] 
```
The initialization should provide a sensible translation of the moving image. 

* With good initialization, begin running the rigid registration, then the affine. Isolate where the registration converges too soon (fails to improve) or diverges (moving image gets further away from the correct solution).

