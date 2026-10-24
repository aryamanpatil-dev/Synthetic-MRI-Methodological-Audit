# 3D Synthetic Brain MRI Evaluation: A Methodological Audit

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![ANTsPy](https://img.shields.io/badge/Registration-ANTsPy-orange)](https://antspy.readthedocs.io/)

This repository contains the official codebase for the paper **"Unmasking Metric Inflation in 3D Synthetic Brain MRI Evaluation: A Forensic Methodological Audit with Coordinate, Masking, Aggregation, and Negative-Control Guard-Rails"**[cite: 172]. 

It provides a robust, fully verified evaluation framework to assess the anatomical fidelity and memorization risk of 3D generative models (VAEs, GANs, Diffusion Models) while preventing common metric inflations, coordinate corruptions, and statistical dependencies[cite: 172].

## Key Features & Mitigated Failure Modes

This pipeline systematically corrects six critical failure modes common in medical imaging evaluation[cite: 177]:

*   **Intensity-Scale Harmonization:** Prevents false-positive distributional rejections by applying within-mask Z-score standardization prior to Kolmogorov-Smirnov testing[cite: 177].
*   **Coordinate Axis Alignment:** Utilizes `dicom2nifti` for mathematically safe LPS-to-RAS conversion, preventing `InstanceNumber` sorting errors that collapse 3D volumes into 2D slivers[cite: 177].
*   **Native Anatomical Masking:** Generates MNI152 brain masks natively within `ANTs` to guarantee perfect affine alignment, preventing cross-library misalignments that clip cortical tissue[cite: 177].
*   **Mask-Restricted Metric Computation:** Computes 3D SSIM and NCC strictly within the foreground brain mask, eliminating the massive artificial inflation caused by empty background voxels (which otherwise comprise ~75% of the bounding box)[cite: 177].
*   **Unbiased Aggregation:** Uses out-of-sample mean similarity for population fidelity instead of nearest-neighbor (max) aggregation, eliminating extreme-value selection bias[cite: 177].
*   **Disjoint Conditional Inference & Stress Testing:** Implements a strict disjoint reference/query protocol (20-reference / 20-query) with plus-one Monte Carlo permutation testing and bootstrapping[cite: 175]. Includes a negative-control pipeline to definitively flag collapsed checkpoints and contaminated cohorts[cite: 177].

## Installation

Clone the repository and install the required dependencies. We recommend using a virtual environment.

```bash
git clone [https://github.com/yourusername/Synthetic-MRI-Methodological-Audit.git](https://github.com/yourusername/Synthetic-MRI-Methodological-Audit.git)
cd Synthetic-MRI-Methodological-Audit
pip install -r requirements.txt
```

**Core Dependencies:**
*   `antspyx`
*   `dicom2nifti`
*   `nibabel`
*   `scikit-image`
*   `scipy`
*   `numpy`

## Repository Structure

```text
├── data/
│   ├── raw/                 # Put your raw DICOM (real) and NIfTI (synthetic) files here
│   └── checkpoints/         # Pre-computed similarity matrices and MNI masks
├── src/
│   ├── harmonization.py     # LPS->RAS conversion, ANTs registration, Z-score scaling
│   ├── metrics.py           # Mask-restricted SSIM and NCC computation
│   ├── inference.py         # Disjoint conditional Monte Carlo permutation & bootstrap
│   └── stress_test.py       # Negative-control mock-checkpoint generation (AutoencoderKL)
├── notebooks/
│   ├── 1_Data_Harmonization.ipynb
│   ├── 2_Metric_Computation.ipynb
│   └── 3_Statistical_Inference.ipynb
├── requirements.txt
└── README.md
```

## Quick Start

**1. Harmonize and Register Data**
Convert raw DICOMs, extract brain masks via ANTs, and register all volumes to MNI152 space.
```bash
python src/harmonization.py --real_dir data/raw/real/ --synth_dir data/raw/synth/ --out_dir data/checkpoints/
```

**2. Compute Mask-Restricted Metrics**
Generate the $S_{RR}$ (Real-to-Real) and $S_{SR}$ (Synth-to-Real) similarity matrices.
```bash
python src/metrics.py --ckpt_dir data/checkpoints/ --metric ssim ncc
```

**3. Run Disjoint Conditional Inference**
Execute the 10,000-iteration Monte Carlo permutation test and reference-split sensitivity analysis.
```bash
python src/inference.py --ckpt_dir data/checkpoints/ --n_perms 10000 --seed 42
```

## Reporting Checklist

To ensure reproducibility and rigorous evaluation in your own 3D synthetic neuroimaging studies, please refer to the **10-Point Reporting Checklist** provided in Table VI of our manuscript[cite: 180]. Key recommendations include reporting exact affine matrices, utilizing disjoint reference cohorts, and always testing pipelines against a known negative control (unstructured noise)[cite: 180].

## Citation

If you use this evaluation pipeline or find the methodological audit useful in your research, please cite:

```bibtex
@article{patil2026unmasking,
  title={Unmasking Metric Inflation in 3D Synthetic Brain MRI Evaluation: A Forensic Methodological Audit with Coordinate, Masking, Aggregation, and Negative-Control Guard-Rails},
  author={Patil, Aryaman},
  journal={arXiv preprint},
  year={2026}
}
```
