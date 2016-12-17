the resolution of the warp field is that of the fixed image space, assuming you run some full resolution iterations.

the scale of the features used to determine the mapping are impacted by the smoothing parameters for syn ( default is 3, 0 which means smooth the field at scale of roughly 3 voxels ) and the neighborhood correlation radius ( usually 4 which indicates a 9x9x9 feature window ).

the effective resolution would be determined by how these 3 things interact : resolution of image wrt real anatomy, smoothing of warp field, neighborhood correlation window.   it would be an interesting exercise to quantify the effective scale with synthetic data.

note that [BSplineSyN](http://journal.frontiersin.org/article/10.3389/fninf.2013.00039/full) will have a different effective resolution than "original syn".  

if one wants to "match" resolutions across two different studies, the easiest way is to acquire data at a resolution that is similar.  e.g.  baby brains are about one third the size of adult brains ( roughly 400cm^3 vs 1200cm^3 ).  so, one might want to acquire data at 0.7mm^3 for the babies to compare to 1mm^3 data for adults.   Why?  because 0.7^3 \approx 0.33 which would assign roughly similar resolution to each anatomical element in babies vs adults.   syn ( with the same parameters ) would then operate at a similar effective scale in both cases.