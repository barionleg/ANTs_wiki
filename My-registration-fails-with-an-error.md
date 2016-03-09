# Common error messages

## Similarity metric errors

```
  Description: itk::ERROR: MattesMutualInformationImageToImageMetric(0x1d1d9ac0): Joint PDF summed to zero
```

Usually seen during the rigid / affine stage, when using a mutual information metric. Other metrics may produce similar errors. 

This means the fixed and moving images don't overlap in physical space, and therefore the similarity metric can't be computed.

Solution: Use an initial transform, see the `-r` option to antsRegistration.

```
 MattesMutualInformationImageToImageMetricv4(0x7feef3eef3f0): Too many samples map outside moving image buffer.
```

The mutual information metric samples a set of points. The exact sampling strategy is determined by the user with the `-m` option. If too many of these points are outside the moving image, this error is generated.

Causes include insufficient overlap between the two images, or an overly restrictive mask. Voxels masked as background are considered outside the image for the purposes of this error.

If this happens partway into registration, it suggests something is going wrong and the images are diverging.

## Memory errors

```
Description: itk::ERROR: [details omitted]
Failed to allocate memory for image.
```

Sometimes also results in a segmentation fault and exit code 139.

If registration quits suddenly with no error message, memory is often the culprit. Some systems have hard limits and jobs that exceed the limits are killed before being able to throw an exception. If you run with verbose output `- v 1`, you can see where the error happens. It is often between stages of registration as the images get larger and thus require more memory.

On shared computing platforms, it's often possible to allocate more RAM. Alternatively configure ANTs to use less by using single precision floats for computation, with `--float 1`.


