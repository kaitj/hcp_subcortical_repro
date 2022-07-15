# HCP Subcortical Connectome Processing Steps

_Details of preprocessing performed as part of the HCP preprocessing pipeline can be found [here](https://www.sciencedirect.com/science/article/pii/S1053811913005053). For further information regarding the diffusion processing, see [Sotiropoulos et al. (2013)](https://www.sciencedirect.com/science/article/pii/S105381191300551X)._

## BigBrain subcortical structures 

Subcortical parcellations in MNI2009b space can be found in the following repository: https://osf.io/xkqb3/. 

1. Transformation between MNI2009b and MNI152NLin6Asym spaces (used by the HCP) was performed using ANTs' `antsApplyTransforms` with **NearestNeighbor** interpolation.

1. From MNI152NLin6Asym space, transformation to individual subject space was performed using HCP provided transforms (`standard2acpc_dc.nii.gz`), again with **NearestNeighbor** interpolation. FSL's `applywarp` was used, with the subject's T1w image used as reference

1. The transformed parcellations were binarized, and the brain stem (also binarized) was added using `fslmaths`. This mask is later used to to create a convex hull.

## Freesurfer ([Scripts](https://github.com/kaitj/dbsc/tree/main/dbsc/resources/example_scripts/freesurfer))

Freesurfer (v7.1) is used to perform parcellation of the thalamus. 

_Standard Freesurfer processing is already performed as part of the HCP pipeline_

1. Using the existing Freesurfer outputs, the `segmentThalamicNuclei.sh` script was used to perform parcellation of the thalamus with default parameters. 

1. The thalamus parcellations were converted from `.mgz` to `.nii.gz` using `mri_convert`, and transformed to MNI162NLin6Asym space using `antsApplyTransforms` with **MultiLabel** interpolation.

## Updating subcortical structures ([Scripts](https://github.com/kaitj/dbsc/tree/main/dbsc/resources/example_scripts/zona_bb_subcortex))

Thalamus structure (whole label) was replaced with the parcellated version of the thalamus. Label updating was performed using `fslmaths`

1. Removed existing thalamus label from segmentation and updated the existing labels. This allows for the new parcellated thalamus labels to be added to the end in numerical order.

1. Parcellated thalamus structure labels are added to the existing labels.

1. Subcortical structures are binarized with the brain stem once again to ensure an accurate mask is created.

1. A convex hull is created from the binarized mask and inverted to be used as an exclusion mask later on.

## Tractography processing ([Scripts](https://github.com/kaitj/dbsc/tree/main/dbsc/resources/example_scripts/mrtpipelines))

_Processing is performed using Mrtrix3 (v3.0_RC3). All `.nii.gz` files are converted to `.mif` files (Mrtrix3 extension) during this process._

1. Subject-specific response functions are estimated from the diffusion data (`dwi2response`) using the `dhollander` algorithm. An average response (`average_response`) is estimated (if test-retest, from a single session) using response functions from all subjects within the dataset.

1. Fibre orientation distributions (FODs) are computed `dwi2fod` for each subject from the average response function and subject-specific diffusion data, using the HCP provided brain mask, all 3 available diffusion shells (1000, 2000, 3000) and the multi-shell, multi-tissue CSD algorithm (`msmt_csd`).

1. Computed FODs are normalised (`mtnormalise`), again using the brain mask and default parameters (max number of iterations (`-niter`): 15, max order: 3, reference value: 0.282095)

1. Tractography was performed (`tckgen`) from the normalised FOD, with a step-size of 0.35mm (1/4 x voxel size), using the inverted convex hull as an exclusion mask, and the updated subcortical segmentations as an inclusion mask. The brain mask was used, both as a mask, and as the seed image. Tracking was performed until 20 million streamlines was selected.

1. Spherical-deconvolution informed filtering (`tcksift2`) was performed on the generated tractography, using both the tractography file and the normalized FODs, outputting weights for each streamline.

1. Streamlines within a 1.5mm radius (`-assignment_radial_search`) are assigned to each label of the subcortical segmentation to create a connectivity matrix of summed streamline weights (`tck2connectome`), with self connections to zero (`-zero_diagonal`). Further, streamline assignments are saved (`-out_assignments`) for future extraction for connections.

1. Each connection is extracted (`connectome2tck`) with associated weights (`-prefix_tck_weights_out`) using the previously saved streamline assignments. 

1. Extracted connections are filtered (`mrcalc`) to remove streamlines passing through subcortical structures (other then expected terminal structures). Weights for each connection are updated to remove the streamlines filtered out (`tckedit`)

1. Filtered connections are combined into a single tractography file (`tckedit)`, and an updated connectivity matrix is computed (`tck2connectome`). Similar to before, a radius of 1.5mm is used to assign streamlines to subcortical segmentations and streamline assignments are once again saved.

## Analysis

Following processing, analysis was performed using the notebooks incuded in this repository.
