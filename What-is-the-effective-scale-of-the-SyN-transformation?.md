the resolution of the warp field is that of the fixed image space, assuming you run some full resolution iterations.

the scale of the features used to determine the mapping are impacted by the smoothing parameters for syn ( default is 3, 0 which means smooth the field at scale of roughly 3 voxels ) and the neighborhood correlation radius ( usually 4 which indicates a 9x9x9 feature window ).

the effective resolution would be determined by how these 3 things interact : resolution of image wrt real anatomy, smoothing of warp field, neighborhood correlation window.   it would be an interesting exercise to quantify the effective scale with synthetic data.