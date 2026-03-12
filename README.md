# Iris Recognition with Adaptive Elliptical Mesh Normalization (AEMN)

This repository contains a complete implementation of four iris normalization methods:
- **Daugman's rubber‑sheet model** (circular)
- **Circular Adaptive** (radial sampling based on edge density)
- **Elliptic Uniform** (elliptical boundaries with uniform sampling)
- **AEMN (Adaptive Elliptical Mesh Normalization)** – a novel method that adaptively adjusts radial sampling density according to local iris texture.

The project includes a full evaluation pipeline: segmentation, normalization, feature extraction (log‑Gabor filters → binary iris codes), and matching via Hamming distance. Extensive experiments (identification, verification, cross‑eye challenge, ablation, timing, and statistical significance) are implemented, reproducing the structure of a typical research paper.

> **Note:** On the MMU Iris Database, the system did not achieve state‑of‑the‑art performance. This is attributed primarily to segmentation challenges. The code is provided as a transparent baseline and a learning resource for anyone interested in iris recognition.

---

## Table of Contents
- [Overview](#overview)
- [Methods](#methods)
- [Dataset](#dataset)
- [Evaluation Protocol](#evaluation-protocol)
- [Results Summary](#results-summary)
- [Lessons Learned](#lessons-learned)
- [Requirements](#requirements)
- [Usage](#usage)
- [Repository Structure](#repository-structure)
- [License](#license)
- [Citation](#citation)

---

## Overview

Iris recognition is one of the most accurate biometric modalities, but its performance heavily depends on reliable segmentation and invariant feature extraction. This project implements four normalization techniques and compares them under identical conditions. The main contribution is **AEMN**, which uses edge saliency to vary the number of radial samples along each angular column, aiming to preserve discriminative texture in high‑information regions.

Despite careful implementation, the overall recognition accuracy remained low on the MMU dataset, highlighting the critical role of segmentation and the difficulty of visible‑wavelength iris recognition.

---

## Methods

### 1. Segmentation
- **Ellipse detection** using adaptive thresholding and contour analysis.
- Fallback to circular Hough transform when ellipse fitting fails.
- Segmentation statistics (pupil radius, iris radius, pupil/iris ratio) are saved.

### 2. Normalization
All methods map the annular iris region to a fixed‑size rectangular strip of **64×360 pixels**.
- **Daugman**: Circular boundaries, uniform radial sampling.
- **Circular Adaptive**: Circular boundaries, radial sampling density modulated by local edge strength.
- **Elliptic Uniform**: Elliptical boundaries (fitted to pupil and iris), uniform radial sampling.
- **AEMN**: Elliptical boundaries, adaptive radial sampling based on edge density (similar to circular adaptive but with elliptical geometry).

### 3. Feature Extraction
- A bank of **log‑Gabor filters** (multiple scales and orientations) is applied in the frequency domain.
- Phase response is quantized to two bits per filter, producing a binary iris code.
- A noise mask excludes the top/bottom 10% of the normalized strip (simulating eyelid occlusion).

### 4. Matching
- **Hamming distance** is used to compare two iris codes, considering only bits where both masks are valid.

---

## Dataset

The primary dataset used is the **MMU Iris Database** (available from [here](http://www.cs.princeton.edu/~andyz/irisrecognition/)). It contains images from 100 subjects, with left and right eye images. The dataset is challenging due to varying illumination, specular reflections, and occasional off‑axis gaze.

**Segmentation statistics on MMU** (after our ellipse detection):
| Metric           | Mean  | Std   |
|------------------|-------|-------|
| Pupil radius (px)| 16.3  | 8.3   |
| Iris radius (px) | 40.9  | 29.4  |
| Pupil/Iris ratio | 0.54  | 0.27  |

The large standard deviation and extreme values (iris radius as low as 8 px) indicate frequent segmentation failures.

---

## Evaluation Protocol

The code reproduces all experiments from a typical research paper:

### Identification
- 5‑fold cross‑validation with a nearest‑neighbor classifier (using Hamming distance).
- Metrics: Accuracy, Rank‑1, Rank‑5, and CMC curve.
- Confidence intervals (95%) and paired t‑tests between methods.

### Verification
- All pairwise comparisons between images.
- Metrics: EER (Equal Error Rate), decidability index (d′), ROC curve, DET curve.
- Score distribution histograms.

### Cross‑Eye Challenge
- Train on left eye images, test on right eye images (and vice versa).
- Measures how well the system generalises across eyes of the same subject.

### Ablation Study
- (Placeholder) In the original code, this involved fine‑tuning a ResNet on the normalized images. Here we note that feature extraction is fixed; ablation could be extended.

### Timing
- Average execution time per image (segmentation + normalization + feature extraction) for each method.

### Visualization
- Segmentation overlays, sample normalized strips, t‑SNE of filter responses, confusion matrices.

---

## Results Summary

All methods performed similarly, with accuracy around **9–10%** (chance level ≈ 1%). Verification EER was **~0.47** (random guess is 0.5), and decidability was **~0.1** (a good system has d′ > 3). The cross‑eye accuracy was also near chance (~5%).

These results indicate that the pipeline, as implemented, does not produce discriminative features. The primary reason is **unreliable segmentation**: if the pupil and iris boundaries are incorrectly located, the normalized strip does not contain genuine iris texture, and any subsequent feature extraction fails.

**Statistical significance**: No method significantly outperformed the others (p > 0.05 in paired t‑tests).

---

## Lessons Learned

1. **Segmentation is the bottleneck** – In iris recognition, accurate localisation of the pupil and iris is far more important than the choice of normalization or feature extraction. Our ellipse‑fitting approach, while robust in ideal conditions, is insufficient for the variability present in MMU.

2. **Generic features are not enough** – Pre‑trained ResNet features (used in the original code) are not designed for iris texture. Switching to log‑Gabor filters is a step in the right direction, but they still require well‑normalized input.

3. **Visible‑wavelength iris recognition is hard** – MMU contains visible‑light images with reflections and occlusions. Near‑infrared datasets (e.g., CASIA) are easier and would likely yield much higher accuracy.

4. **AEMN potential remains untested** – Because the common segmentation step failed, the adaptive sampling of AEMN could not demonstrate its advantage. A fair comparison would require a reliable segmentation method (e.g., a deep learning model) applied to all images first.

5. **Documentation and transparency matter** – Sharing a “failed” experiment helps others avoid the same mistakes and provides a baseline for future improvements.

---

## Requirements

- Python 3.7+
- OpenCV
- NumPy
- SciPy
- scikit‑learn
- scikit‑image
- Matplotlib
- Seaborn
- tqdm
- pandas
- (optional) Google Colab (for mounting Drive)

Install with:
```bash
pip install opencv-python numpy scipy scikit-learn scikit-image matplotlib seaborn tqdm pandas


## Citation

Oyeniran, O. (2026). Iris Recognition with Adaptive Elliptical Mesh Normalization. GitHub repository: https://github.com/yourusername/IrisRecognition_AEMN
