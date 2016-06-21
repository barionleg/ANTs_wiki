A detailed walkthrough of an example ANTs registration, with options explained in detail.

```
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
