*by Dorian Pustina and Philip Cook*  
A walkthrough of an example ANTs registration, with options explained in detail:

```bash
antsRegistration --dimensionality 3 --float 0 \  
        --output [$thisfolder/pennTemplate_to_${sub}_,$thisfolder/pennTemplate_to_${sub}_Warped.nii.gz] \  
        --interpolation Linear \  
        --winsorize-image-intensities [0.005,0.995] \  
        --use-histogram-matching 0 \  
        --initial-moving-transform [$t1brain,$template,1] \  
        --transform Rigid[0.1] \  
        --metric MI[$t1brain,$template,1,32,Regular,0.25] \  
        --convergence [1000x500x250x100,1e-6,10] \  
        --shrink-factors 8x4x2x1 \  
        --smoothing-sigmas 3x2x1x0vox \  
        --transform Affine[0.1] \  
        --metric MI[$t1brain,$template,1,32,Regular,0.25] \  
        --convergence [1000x500x250x100,1e-6,10] \  
        --shrink-factors 8x4x2x1 \  
        --smoothing-sigmas 3x2x1x0vox \    
        --transform SyN[0.1,3,0] \  
        --metric CC[$t1brain,$template,1,4] \  
        --convergence [100x70x50x20,1e-6,10] \  
        --shrink-factors 8x4x2x1 \  
        --smoothing-sigmas 3x2x1x0vox \  
        -x $brainlesionmask
```




# INTRODUCTION:  
Registration (sometimes called "Normalization") brings one image to match another image, such that the same voxels refers roughly to the same structure in both brains. Often the fixed (or target) image is a template, but you can register any two images with each other, there is nothing special in a template from the computation perspective.  
  
### Registration algorithms:  
To achieve optimal registration, a set of algorithms are used in ANTs: rigid, then affine, then SyN. A "rigid" registration does not deform or scale the brain, it only rotates and moves it (the whole image is considered a rigid object). An "affine" registration allows not only rotation and motion, but also shearing and scaling. This further allows to match brains in size. The "rigid" and "affine" registrations are also called linear registrations, because each point of the image depends on the motion in space of the other points. Linear registrations help only superficially, you can imagine that gyri and sulci still remain misaligned. To achieve proper registration of individual gyri and sulci we need  non-linear registrations. There are several non-linear algorithms in ANTs, but we typically use SyN, one of the top  performing algorithms (see [Klein 2009](http://www.ncbi.nlm.nih.gov/pubmed/19195496)).   

### Iterations:  
Beside going through registration algorithms, ANTs improves the registration within each algorithm gradually at different resolutions, called levels. The idea is to start with "blurry" images (low resolution, highly smoothed), register those to each other, then go to the next step, with a sharper higher resolution version, and so on. This is the reason why you see 1000x500x250x100 in registration calls, it means there will be 4 levels. The numbers show how many iterations for each level.  
The above antsRegistration call runs a rigid, an affine, and a SyN registration. It is the exact call made by the script antsRegistrationSyN.sh. The following is a line by line comment of what each line of the call does. Note that words starting with $ are variables we have defined before. I.e.:  
thisfolder=/user/local/mydata/  
sub=Subject1  
template=Template.nii.gz  
t1brain=Subject1.nii.gz  

***
## Preprocessing options

> The first two arguments tells `antsRegistration` that the images are 3D, and double precision floating point numbers will be used for computation and output.
  
		antsRegistration --dimensionality 3 --float 0 \  
|  
  
> save transformation matrices with prefix $thisfolder/pennTemplate_to_${sub}_  
> save the moving image resampled into the fixed space as $thisfolder/pennTemplate_to_${sub}_Warped.nii.gz
> Optionally, one can also output the fixed image resampled into the moving space.  
		--output [$thisfolder/pennTemplate_to_${sub}_,$thisfolder/pennTemplate_to_${sub}_Warped.nii.gz] \
|  
  
> The interpolation type used when writing the warped image(s).  
> This only applies to the resampling of the output images (specified with `--output`) at the end of the registration. It does not affect resampling internally during registration.
  
		--interpolation Linear \
|  
  
> Winsorization clips outlier intensities during registration. The specified quantiles of the image histograms define the minimum and maximum intensities. 
> This can help stabilize image metrics, but overly aggressive clipping can destroy contrast in the image.  
> The output warped images are not affected by this setting.
> To preview winsorization on an image, use `ImageMath` with `TruncateImageIntensity`. 
  
		--winsorize-image-intensities [0.005,0.995] \
|  
  
> Histogram matching is a pre-processing step, transforming the input intensities such that the histogram of the fixed and moving image are matched as closely as possible. This can help in certain applications, but is off by default.
  
		--use-histogram-matching 0 \
|  
  
> An initial moving transform is used to quickly align the images before registration. The optimization process requires images to roughly overlap and will fail if images are too far apart in physical space. There are various options for the initialization:
> 0-match by mid-point (i.e., center voxel of one image will be brought in line with center voxel of the other)  
> 1-match by center of mass  
> 2-match by point of origin (i.e. physical coordinates 0,0,0, as defined by the image headers)
> The center of mass alignment usually works well.
  
> Alternatively, you can specify an initial affine transform .mat file obtained with `antsAI`, which runs many quick registrations with different initial transforms (rotations and / or translations), to find what works best. 
 
		--initial-moving-transform [$t1brain,$template,1] \
|  
  
## Registration stages

Each stage of registration has a transformation model, one more more similarity metrics, and details of how to run the optimization. The stages proceed from simplest to most complex transforms. Most users will start with Rigid.


The gradient step for the Rigid transform is 0.1. As with all optimization, the step size involves multiple tradeoffs. Smaller steps can be more accurate but take longer, and may appear to converge prematurely. Larger steps can overshoot the best solution. For most brain registration, optimal values are around 0.1-0.25.
  
        --transform Rigid[0.1] \
|  
  
> The metric measures similarity between two images, and the gradient of the metric informs the update of the transform parameters. All ANTs metrics have the form `[fixed, moving, parameters]`. It's important to keep track of which space define as the fixed and which as the moving.
>Mutual information uses the histograms of the two images to check the similarity, meaning it can detect similar anatomical patterns even if the images do not correlate well. This makes it very useful for inter-modality registration. It is also fairly fast to compute and robust, making it a good choice for rigid registration. The value of 1 is a weight, used if you do multimodal registration. Here is an example of a multimodal registration call  
		# --metric CC[$t1brain,$template,0.5,4] # CC radius 4, weight 0.5 on t1, dense sampling 
		# --metric MI[$t2brain,$T2template,0.5,32,Regular,0.25] # weight 0.5 on t2  
		# MI parameters are [fixed, moving, weight, bins, sampling, samplingPercentage] 
The weights are normalized internally, so they sum to 1. The metrics themselves are also normalized so you can combine different metrics without any correspondence between the raw metric numbers. In the above example, CC on t1 and MI on t2 are weighted equally, so they will both contribute equally to the transform.

Our example call has 32 bins for MI, and values are sampled regularly in 25% of the voxels, i.e. one voxel every four is considered. 
  
        --metric MI[$t1brain,$template,1,32,Regular,0.25] \
|  
  
> we will run 4 levels (multi-resolution steps) with a maximum number of iterations of 1000,500,250,100. The threshold (1e-6) tells the algorithm to stop if the improvement in mutual information has not changed more than 1e-6 in the last 10 iterations (convergenceWindowSize=10). To translate it in plain english:  
"if the change in MI value for the last 10 iterations is below 1e-6, stop the iterations and go to next level". Because of this test, we can run a large number of iterations for the initial levels (here 1000). The images here are usually smooth and quite small, as explained below, and will almost always converge quickly.
   
        --convergence [1000x500x250x100,1e-6,10] \
|  
  
> the 4 hierarchical steps will have resolutions divided by 8,4,2,1. The number of "shrink factors" must match the number of levels in the `--convergence` option. The factors use the fixed image as reference. If the fixed image has spacing 1x1x1mm, and the moving image has spacing 2x2x2mm, then `--shrink-factors 4x2x1` will register images at 4mm resolution, then 2mm, then 1mm. But if you switch the fixed and moving images, `--shrink-factors 4x2x1` will register images at 8mm, then 4mm, then 2mm. For 3D images, shrinking the volumes by a factor of 2 decreases the number of voxels by a factor of 8. So running iterations at shrink factor 1 requires about 8 times as much computation as at shrink factor 2, and so on.
  
        --shrink-factors 8x4x2x1 \
|  
  
> Next are the smoothing values for each step. The smoothing is Gaussian with a kernel standard deviation of 3,2,1,0 voxels. Here they are specified in voxels, but they can also be in mm.  To convert the sigma to FWHM you can use roughly a factor of 2.36. The images are smoothed before being downsampled at each level. The number of smoothing levels must match the number of iteration levels `--convergence`.
  
        --smoothing-sigmas 3x2x1x0vox \
|  
  
> END OF RIGID STAGE
> ###########################################  
  
        
|  
  
> ###########################################  
AFFINE STAGE  
the speed of deformation (gradientStep) at each iteration is again 0.1  
  
        --transform Affine[0.1] \
|  
  
> all following options are explained above
  
        --metric MI[$t1brain,$template,1,32,Regular,0.25] \
        --convergence [1000x500x250x100,1e-6,10] \
        --shrink-factors 8x4x2x1 \
        --smoothing-sigmas 3x2x1x0vox \
|  
  
> END AFFINE STAGE  
############################################  
  
|  
  
>############################################  
START THE THIRD STAGE: SyN  
The parameters inside SyN[] are: gradientStep,updateFieldVarianceInVoxelSpace,totalFieldVarianceInVoxelSpace  

`gradientStep` - tells the algorithm how much each point can move after each iteration. The SyN metric computes in which direction each point needs to move. This movement can be large (high gradientStep) or small (low gradientStep). Optimal values for brain imaging are usually in the range of 0.1-0.25. 

After each iteration, a gradient field is computed, which indicates how each point within the image will shift in space. This small deformation (or "updated" gradient field) is combined with previous updates to form a "total" gradient deformation. Both the update field and the gradient field can be smoothed to regularize the deformation.
  
`updateFieldVarianceInVoxelSpace` - By adding a penalty here, we smooth the deformation computed on the "updated" gradient field at each iteration. The default of 3 voxels has been shown to work fairly well. Increasing this will make the update field smoother, meaning it will be less sensitive to local deformation.

`totalFieldVarianceInVoxelSpace` -  By adding a penalty here, we smooth the deformation computed on the "total" gradient field. The smoothing here, therefore, is applied on all the deformations computed from the beginning (i.e., at this and all previous SyN iterations).  
In principle, smoothing of the update field can be viewed as fluid-like registration whereas smoothing of the total field can be viewed as elastic registration. Adding a regularization term to the total field makes the image "stiffer", in that it dampens the total amount of deformation. 

Both of these parameters can impact the optimal step size and number of iterations required for convergence.
  
        --transform SyN[0.1,3,0] \
|  
  
> At the SyN stage, we use cross-correlation (CC) instead of mutual information (MI). For intramodality neuroimaging, CC works better than MI for deformable registration. Instead of checking the histograms of images, CC measures correlation between local neighborhoods of voxels. For each point in the image, CC is computed for the point and its neighbors; a radius of `n` means the neighborhood will be a 2n+1 cube around the point. A large radius therefore increases computation time substantially, but uses more information as a result. A value of 2-4 works well for most brain imaging, larger can help improve robustness for hard problems (eg, low-information neighborhoods where tissue damage or removal has reduced contrast).
 
the call is [fixed,moving,weight,radius]
  
        --metric CC[$t1brain,$template,1,4] \
|  
  
> the following options are explained above, and work the same way. Note that because SyN and CC are quite computationally expensive, the 20 full-resolution iterations here will take the longest. A large number of full-resolution iterations can take longer than all the preceding iterations combined across all stages. With `--verbose`, the registration will print time taken for each iteration.
  
        --convergence [100x70x50x20,1e-6,10] \
        --shrink-factors 8x4x2x1 \
        --smoothing-sigmas 3x2x1x0vox \
|  
  
> END OF SyN TRANSFORMATION  
#############################################  
  
|  
  
  
> A mask restricts computation of the metric to a certain region. The mask should be 1 where similarity should be evaluated, and 0 otherwise. Anything outside the mask does not contribute to the similarity metric. This can help but also hinder registration. It can help by ignoring things that are leading to problems (eg, if we're trying to align brains within a head image, we might not care about the neck so much). But it can also destabilize registration as points moving into and out of the mask can change the metric sharply. Masking out smaller regions to deal with specific problems (eg, lesions) is less likely to cause problems than attempting to draw a mask around the entire structure of interest.
  
        -x $brainlesionmask
|  
> antsRegistration can accept masks on the target, moving, or both images. Sometimes you may need to mask both sides with separate masks. In that case you can use something like this:  
		# -x [$fixed_mask,$moving_mask]
  
  
  
** Tips for registration:**  
1. Run a bias correction before antsRegistration (i.e. N4). It helps getting better registration.   
2. Remove the skull before registering brain images. 
3. Use a mask when there are features in one image that have no proper match to the other image. For example, images with different fields of view, or a brain with lesions being registered to one without. If you really don't have the lesion mask, even a coarse and imprecise drawing of lesions helps (see [Andersen 2010](http://www.ncbi.nlm.nih.gov/pubmed/20542122)).  
4. A good rigid and affine alignment will make deformable registration faster and more robust. If registration is failing, run the rigid part only, then the affine part only. If these are bad, fix them before spending more time on deformable registration.