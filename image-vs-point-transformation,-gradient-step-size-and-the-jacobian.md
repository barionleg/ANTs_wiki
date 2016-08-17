from https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3065962/pdf/nihms238501.pdf

"Due to the linear deformation penalty, this gradient parameter does not typically need to be changed for SyN. Its useful range—for geodesic SyN—is between 0.1 and 1.0 where the optimal value will depend upon the nature of the problem, the regularization choice and the data. For greedy SyN, the useful range is narrower: 0.1 to 0.5 for most problems and for Gauss[3,0] regularization. Increasing the deformation field regularization (a non-zero second parameter) may require increasing the gradient step size. While we have found results to be robust to choices for the gradient descent parameter, values that are too large will result in energy oscillation while values that are too small will result in slow convergence." 

​i should have used stronger language in the discussion above - ie large step sizes will invalidate the algorithm's output.​  this is due to what is known as the CFL (courant friedrichs lewy) condition https://en.wikipedia.org/wiki/Courant–Friedrichs–Lewy_condition which, in our case, suggests we should never select gradient step sizes (much) greater than 1 where 1 is in voxel coordinates.  the gradients upon which our algorithms are based are only valid very locally so trying to step beyond 1 voxel with a local gradient is not recommended.

**Note**: the discussion above refers to the old (not used by us, not much maintained) program `ANTS` - but the same things hold true for `antsRegistration`.  in both cases, regularization is applied to the "update" (velocity) fields and "total" (deformation) fields ... for standard diffeomorphic registration, the update field regularization should be around 3 with zero total field regularization.  in some cases, one may want to increase regularization of the total field ( e.g. if one is dealing with very noisy data ) or use transformation models such as `BSplineSyN` that may provide "smoother" results.  alternatively, one may increase the value from 3 to something greater in order to focus the registration on coarser scale features.  note that there is an interplay between multi-resolution parameters and these scale parameters.

another point of confusion is the jacobian.  it derives from the determinant of the deformation gradient ( see the ants2 pdf documentation which gives more information and examples ).  for morphometry, one should always compute the jacobian from the mapping that has its domain in the template space.  in this way, the collection of jacobian images will be "in the same space".  we have an example here:  https://github.com/stnava/ANTsTutorial/blob/master/src/phantomMorphometryStudy.Rmd that runs in ANTsR and is based on simulated data.

one might learn about these issues - and the relationship between the jacobian of the Warp and the InverseWarp and the volumes of the input images -- by studying the github project :

https://github.com/stnava/jacobianTests

if you clone that project and then run the following :

```
nm=CIRC
dim=3
f=sphere1negheaderRad11.nii.gz
m=sphere1negheaderRad13.nii.gz
antsRegistration -d $dim \
  -m demons[  $f, $m ] \
  -t syn[ 0.10, 3, 0.0 ] \
  -c [100x100x100,1.e-8,20]  \
  -s 2x1x0vox  \
  -f 4x2x1 -l 1 -u 1 -z 1 \
  -o [${nm},${nm}_diff.nii.gz,${nm}_inv.nii.gz]
CreateJacobianDeterminantImage $dim CIRC0Warp.nii.gz /tmp/circAjacobian.nii.gz 0 0
```

then you will see that:

```
circAjacobian.nii.gz
```

is an image with mostly positive values inside the region of the fixed image.

this indicates ( as in ants2.pdf documentation fig 7, fig 21 and fig 22  which cover the jacobian https://github.com/stnava/ANTsDoc ) that the fixed image has to expand to meet the moving image.

also - keep in mind that the values of the jacobian are inverses of each other.  that means that your statistics will be nearly identical (up to sign) regardless of whether you use the jacobian of the forward Warp or InverseWarp **assuming that you have mapped the jacobian derived from the InverseWarp to the template space**.

this is a nice [explanation of the jacobian](http://www.brown.edu/Departments/Engineering/Courses/En221/Notes/Kinematics/Kinematics.htm).  see section 3.5 and 3.6. please also see figure 7 from ants2.pdf 

referencing the figure, if we assign imageA as sphere1Rad11 ​and imageB as sphere1Rad13​ ​then note the figure's comments about the 2nd and third voxels, then it fits exactly what we see in jacobianTests. i.e.  the jacobian values will be overall >1 because the moving image is larger than the fixed. ​in the file `rcalc.R` we see that the error estimates​ are based on looking at the volume of the moving image divided by the volume of the fixed image which also produces a value > 1.  

​so we have, using the ants convention:

```
J(x) \approx VolumeOfTinySphereAtXAfterMappingToMoving / VolumeOfTinySphereAtXInTemplateSpace
```

​so the tiny sphere at x in template space is in the original configuration.  the volume of the tiny sphere changes under the forward mapping applied to that local region at x.   the ratio of these volumes, as above, is a local estimate of jacobian at x.​ so, if we use segmentations as validation (just as we do in the jacobianTest), then one would compute  

```
volume( sphere1Rad13​ ) / volume( sphere1Rad11 )
```

which will be similar to:

```
  \sum_{ x in points in sphere1Rad11} J( \phi( x ) ) 
```

where J is the jacobian determinant computed from phi at x. 

**Note on directionality of the mapping**:   phi is the ForwardWarp which maps points from the fixed space to the moving space.  It is also the mapping that one applies to deform the moving image to the fixed image. this is the difference between mapping points and images, as described in figure 7 of ants2.pdf ( quite an old figure but still correct in its description !).   


**Note on group comparisons with the jacobian:**  Recall that - in a population tensor-based morphometry or jacobian study - one maps all subjects in the population to a group template.  If we do this, then the ANTs way of defining and computing the jacobian will produce *positive* values in a t-test between two cohorts if group A is "bigger" than group B.


