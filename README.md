# MRI Preprocessing Pipeline for Brain Metastasis Tumor Segmentation

A preprocessing pipeline for real clinical brain MRI scans, built to prepare
contrast-enhanced T1 (T1+GD) images and tumor segmentation masks for a deep
learning tumor segmentation and feature extraction project.

## Overview

This pipeline standardizes 44 real patient MRI scans (single timepoint per
patient, selected from a larger multi-timepoint dataset) so they can be used
consistently in downstream segmentation and radiomic feature extraction.

**Pipeline stages:**

| Step | Stage | Purpose |
|------|-------|---------|
| 0 | Patient & scan selection | One contrast-enhanced scan + mask per patient |
| 1 | Orientation standardization | Consistent anatomical orientation (ANTsPy, `IAL`) |
| 2 | Bias field correction | Removes scanner-induced intensity inhomogeneity (N4) |
| 3 | Histogram matching | Normalizes intensity distribution to a reference template |
| 4 | Registration | Deformable (SyN) registration to the MNI152 template |
| 5 | Skull stripping | Deep-learning brain extraction (ANTsPyNet) |

Each stage reads the previous stage's saved output and writes its own —
processing is fully resumable and safe to re-run (already-completed patients
are automatically skipped).

## Data

Sourced from the **MolAB Brain Metastasis Dataset**
([molab.es/datasets-brain-metastasis-1](https://molab.es/datasets-brain-metastasis-1/?type=metasrd)):
45 real clinical patients, multiple MRI timepoints/sequences each, with
radiologist-annotated tumor masks and accompanying clinical/radiomic data.

- **Selection:** one scan per patient (earliest available contrast-enhanced
  scan with a corresponding tumor mask), to avoid data leakage from multiple
  timepoints of the same patient
- **Excluded:** 1 patient (oversized, non-contrast reformatted scan type with
  no valid contrast-enhanced alternative available)
- **Final cohort:** 44 patients, each with one T1+GD image and one
  radiologist-annotated tumor mask

Raw patient data is not included in this repository for privacy reasons.

### Citation

Use of this dataset requires citing the original publication:

> Ocaña-Tienda, B., et al. A comprehensive dataset of annotated brain
> metastasis MR images with clinical and radiomic data. *Scientific Data*
> 10, 208 (2023). https://doi.org/10.1038/s41597-023-02123-0

## Requirements

```
antspyx
antspynet
SimpleITK
pandas
numpy
matplotlib
```

## Template

Registration uses the ICBM 2009a Nonlinear Symmetric MNI template
([source](https://nist.mni.mcgill.ca/icbm-152-nonlinear-atlases-2009/)).

## Acknowledgments

This pipeline was built starting from the techniques demonstrated in
[Angeluz-07/MRI-preprocessing-techniques](https://github.com/Angeluz-07/MRI-preprocessing-techniques),
a free tutorial series on brain MRI preprocessing in Python. The
implementation here has been substantially restructured for batch processing
across a real multi-patient clinical dataset, including:

- Automated patient/scan selection from inconsistent multi-timepoint data
- Full pipeline chaining (each step consumes the previous step's output)
- Crash-safe, resumable batch processing for all patients
- A fix to mask registration to use nearest-neighbor interpolation, preserving
  binary tumor labels (the original tutorial's default linear interpolation
  would blur label boundaries)

## Status

Preprocessing complete and validated (file-count and visual mask-overlay
checks across patients with varying tumor sizes). Next stage: tumor
segmentation.
