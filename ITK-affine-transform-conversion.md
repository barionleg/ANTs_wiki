Taken from [here](https://sourceforge.net/p/advants/discussion/840261/thread/9fbbaab7/?limit=25#1783).

> Dear ANTs experts,
>
> I'm trying to obtain a 4x4 matrix that maps from world-coordinates in moving image to world-coordinates in fixed image after applying a linear ANTs registration.
>
> I'd like to be able to convert the (homogeneous) coordinate set P (nx4 with last entries all ones) to P' by multiplying P*M. And looking to obtain the matrix M.
>
> I checked the ANTs manual and a lot of googling, also played around a lot already but it's hard for me to find out what to do exactly.
>
> When I load an affine transform in Matlab I get the "AffineTransform_float_3_3" and "fixed" variables.
>
> Simply ``mat=[reshape(AffineTransform_float_3_3(1:9),[3,3])',AffineTransform_float_3_3(10:12)];``
>
> Does not do the trick (gives me a matrix that is different from M).
>
> The ANTs manual stays a bit vague in chapter 1.8. Point L is not defined in the text but apparently the center of the rotation. But the matrix composed of a-i is already mixed (all transforms composed).
>
> Any help would be really appreciated! If anyone has a specific formula / script on how to exactly obtain M I'd be really happy – it's just so easy to do something wrong here..
>
> Thanks so much, everyone,
> Andy


Dear Nick & Brian,

thanks again so much for your time & help.

In case someone else will be interested in the same question, I found a solution based on this post:

http://public.kitware.com/pipermail/vtkusers/2013-April/078931.html

The matrix stored in the .mat file produced by ANTs/ITK does map from mm to mm space (i.e. world- not voxel space) but it does need some preparation to get M. Again, M will be the 4x4 matrix that maps from world space of the moving (x,y,z) to world space of the fixed image (x',y',z') by:

```
[x',y',z',1]=M*[x,y,z,1];
```

Inside the .mat file there are two variables, "AffineTransform_float_3_3" [12x1] and "fixed" [3x1].

In the ITK code, the latter is often referred to as "m_Center". The first 9 entries of the former, in square form, are often referred to as "m_Matrix". Finally, there's an "m_Offset" or "offset" variable that needs to be computed from the two variables.

M is being built in this way:
M =  
a b c o  
d e f p  
g h i q  
0 0 0 1  
where a-i are the first 9 entries of "AffineTransform_float_3_3", i.e. "m_Matrix".

The last three entries of the same variable, j,k,l ( := "m_Translation") and the three entries of fixed ( := "m_Center") will be used together with the 9 entries of "m_Matrix" in the way defined in itkMatrixOffsetTransformBase.hxx>ComputeOffset to compute o, p and q.

Finally, when working in RAS space (instead of the LPS ANTs/ITK space) you'd need to sign flip some entries of the matrix
M=M.*...  
[1 -1 1 1  
-1 1 1 1  
1 1 1 -1  
1 1 1 1];  

This last step is probably obsolete when you stay completely within the ITK world.

I empirically validated this on 1000 ground truth header modifications. The solution above seems to work well inside the SPM world. Still, please use with care & validate in your specific setting. As Nick pointed out above, NIfTI headers are treated very differently depending on the software package one uses.

There is also a Matlab function "ea_antsmat2mat" inside our Lead-DBS repository that does the trick and takes the two inputs from the .mat file.

Best, Andy