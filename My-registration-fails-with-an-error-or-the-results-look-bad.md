# Common error messages

```
Description: itk::ERROR: MattesMutualInformationImageToImageMetric(0x1d1d9ac0): Joint PDF summed to zero
```

Usually seen during the rigid / affine stage, when using a mutual information metric. Other metrics may produce similar errors. 

This means the fixed and moving images don't overlap in physical space, and therefore the similarity metric can't be computed.

Solution: Use an initial transform, see the `-r` option to antsRegistration.

