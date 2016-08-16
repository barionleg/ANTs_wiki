from https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3065962/pdf/nihms238501.pdf

"Due to the linear deformation penalty, this gradient parameter does not typically need to be changed for SyN. Its useful range—for geodesic SyN—is between 0.1 and 1.0 where the optimal value will depend upon the nature of the problem, the regularization choice and the data. For greedy SyN, the useful range is narrower: 0.1 to 0.5 for most problems and for Gauss[3,0] regularization. Increasing the deformation field regularization (a non-zero second parameter) may require increasing the gradient step size. While we have found results to be robust to choices for the gradient descent parameter, values that are too large will result in energy oscillation while values that are too small will result in slow convergence." 

​i should have used stronger language in the discussion above - ie large step sizes will invalidate the algorithm's output.​  this is due to what is known as the CFL (courant friedrichs lewy) condition https://en.wikipedia.org/wiki/Courant–Friedrichs–Lewy_condition which, in our case, suggests we should never select gradient step sizes (much) greater than 1 where 1 is in voxel coordinates.  the gradients upon which our algorithms are based are only valid very locally so trying to step beyond 1 voxel with a local gradient is not recommended.

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

