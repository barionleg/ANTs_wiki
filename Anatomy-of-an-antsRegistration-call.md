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
To achieve optimal registration, a set of algorithms are used in ANTs: rigid, then affine, then SyN. A "rigid" registration does not deform or scale the brain, it only rotates and moves it (the whole image is considered a rigid object). An "affine" registration allows not only rotation and motion, but also shearing and scaling. This further allows to match brains in size. The "rigid" and "affine" registrations are also called linear registrations, because each point of the image depends on the motion in space of the other points. Linear registrations help only superficially, you can imagine that gyri and sulci still remain misaligned. To achieve proper registration of individual gyri and sulci we need  non-linear registrations. There are several non-linear algorithms in ANTs, but we typically use SyN, one of the top  performing algorithms (see Klein 2009).   

### Iterations:  
Beside going through registration algorithms, ANTs improves the registration with each algorithm gradually at different levels. The idea is to start with a "blurry" low resolution version, register it to the best value possible, than go to the next step, with a sharper higher resolution version, and so on. This is the reason why you see 1000x500x250x100 in registration calls, it means there will be 4 levels 
The above antsRegistration call runs a rigid, an affine, and a SyN registration. It is the exact call made by the script antsRegistrationSyN.sh. The following is a line by line comment of what each line of the call does. Note that words starting with $ are variables we have defined before. I.e.:  
thisfolder=/user/local/mydata/  
sub=Subject1  
template=Template.nii.gz  
t1brain=Subject1.nii.gz  



***
## Inline code explained  
> first two arguments tells the images are 3D, no floating point will be use (double instead)  
  
		antsRegistration --dimensionality 3 --float 0 \  
|  
  
> save transformation matrices with prefix $thisfolder/pennTemplate_to_${sub}_  
> save registered image as $thisfolder/pennTemplate_to_${sub}_Warped.nii.gz
  
		--output [$thisfolder/pennTemplate_to_${sub}_,$thisfolder/pennTemplate_to_${sub}_Warped.nii.gz] \
|  
  
> The interpolation type used when saving the warped image.  
> Applies just to the output image (from moving), nothing else
  
		--interpolation Linear \
|  
  
> deal with outlier voxels. Clips values <5/1000 and >995/1000.  
> Range can be restricted for bad images, but be careful because it may destroy contrast in image.  
> This helps because images may have a few voxels with high value that impact badly the registration
  
		--winsorize-image-intensities [0.005,0.995] \
|  
  
> boolean if histogram of input and output images should be matched.  
> If registering T1 on T2 is not good to set to 0, because they are not in same modality.
  
		--use-histogram-matching 0 \
|  
  
> registration works in real coordinates given by the scanner. So images can start quite far from each other (e.g., one in New York, one in London). An initial move is required to bring the images roughly in the same space (close to each other)  
> options are:  
> 0-match by mid-point (i.e., center voxel of each image  
> 1-match by center of mass  
> 2-match by point of origin (i.e. coordinates 0,0,0)  
> One can also point to a .mat file obtained with antsAI, but there is no need. The AI solution is implemented in antsCortThicknes.sh and runs an affine with several random changes to check if one "unsual" solution is better. Useful if there are strong orientation issues.  
> the command tells [fixed,moving,option]
   
		--initial-moving-transform [$t1brain,$template,1] \
|  
  
> ####################################  
> THIS IS THE FIRST TRANSFORMATION: RIGID  
step size is 0.1, how fast you go with changes in the image transformation. Too big and you may overshoot. Too long and you wait longer.
  
        --transform Rigid[0.1] \
|  
  
> mutual information measures how similar the two images look. It uses the histograms of the two images to check the similarity. The histogram will have 32 bins, and values are sampled regularly at 25%, i.e. a voxel is considered every four.  
The value of 1 is a weight used if do multimodal registration. Here is an example  
		# --metric MI[$t1brain,$template,0.7,32,Regular,0.25] # weight 0.7 on t1  
		# --metric MI[$t2brain,$T2template,0.3,32,Regular,0.25] # weight 0.3 on t2  
		# the call format is [fixed, moving, weight, bins, sampling]  
  
        --metric MI[$t1brain,$template,1,32,Regular,0.25] \
|  
  
> we will run 4 levels (or multi-resolution steps) with a maximum number of iterations of 1000,500,250,100. The threshold (1e-6) tells the algorithm to stop if the improvement in mutual information has not changed more than 1e-6 in the last 10 iterations (convergenceWindowSize=10). To translate it in plain english:  
"if the change in MI value for the last 10 iterations is below the 1e-6, stop the iterations and go to next level"  
Typically not all iterations are run and the loop finishes because no further improvement is possible. If you want to run all iterations, set the threshold to a large negative number. This will allow iterations to keep going even if mutual information becomes worse.  
  
        --convergence [1000x500x250x100,1e-6,10] \
|  
  
> the 4 hierarchical steps will have resolutions divided by 8,4,2,1  
for an image with 256x256x256 voxels, the levels will work at 32mm, 64mm, 128mm, 256mm
  
        --shrink-factors 8x4x2x1 \
|  
  
> Here are the smoothing values for each step: sigma 3,2,1,0.  
To convert the sigma in amount of mm you can use roughly a factor of 2.36. The above correspond to 7mm, 5mm, 2mm and no smoothing  
Need to check if the factor is correct. ANTs uses simple Gaussian, not FWHM  
Note, smoothing occurs before shrinking to lower resolution.  
Phil, sigmas are in vox here, why is that?  
  
        --smoothing-sigmas 3x2x1x0vox \
|  
  
> END OF RIGID TRANSFORMATION  
> ###########################################  
  
        
|  
  
> ###########################################  
THIS IS THE SECOND TRANSFORMATION: AFFINE  
the speed of change each iterations (step size) is again 0.1  
  
        --transform Affine[0.1] \
|  
  
> all following options are explained above
  
        --metric MI[$t1brain,$template,1,32,Regular,0.25] \
        --convergence [1000x500x250x100,1e-6,10] \
        --shrink-factors 8x4x2x1 \
        --smoothing-sigmas 3x2x1x0vox \
|  
  
> END AFFINE TRANSFORMATION  
############################################  
  
|  
  
>############################################  
THIS IS THE THIRD TRANSFORMATION: SyN  
  
        --transform SyN[0.1,3,0] \
|  
  
> here we use cross-correlation (CC) instead of mutual information (MI).  
Instead of checking the histograms of images, now we check individual voxels. A radius of 4 indicates that for each voxel we take 4 layers around it (728 voxels) and compute the correlation with the respective 728 on the other image. So, now we care that the surrounding neighborhood of voxels are as similar as possible.  
the call is [fixed,moving,weight,radius]
  
        --metric CC[$t1brain,$template,1,4] \
|  
  
> the following options are explained above
  
        --convergence [100x70x50x20,1e-6,10] \
        --shrink-factors 8x4x2x1 \
        --smoothing-sigmas 3x2x1x0vox \
|  
  
> END OF SyN TRANSFORMATION  
#############################################  
  
|  
  
> mask defined in template (aka target) space. It will restrict all computations only to voxels with value 1 (i.e. brain), and ignore whatever is outside the mask.  
 why in template space? because you are supposed to move the image in that space and check how well it fits with your target.  
this is a problem for patients with lesions: the lesion is drawn on the subject's image.  
don't worry, just flip the order, make your subject fixed and move the template on it.  
This is the reason why all above calls show template as moving
  
        -x $brainlesionmask
|  
  
  
** Tips for registration:**  
1. Run a bias correction before antsRegistration (i.e. N4). It helps getting better registration.   
2. Remove the skull before antsRegistration. If you have two brain-only images, you can be sure that surrounding tissues (i.e. the skull) will not take a toll on the registration accuracy.  
3. Never register a lesioned brain with a healthy brain without a proper mask. The algorithm will just pull the remaining parts of the lesioned brain to fill "the gap". Despite initial statements that you can "normalize" lesioned brains drawing the lesion, there is evidence showing that results are sub-optimal. If you really don't have the lesion mask, even a coarse and imprecise drawing of lesions helps (see [Andersen 2010](http://www.ncbi.nlm.nih.gov/pubmed/20542122)).  
3. Don't forget to read the parts of the [manual](http://stnava.github.io/ANTsDoc/) related to registration.  