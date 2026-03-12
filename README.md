# Adaptive Elliptical Mesh Normalization (AEMN) for Robust Iris Texture Preservation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository contains the implementation of four iris normalization methods, including a novel **Adaptive Elliptical Mesh Normalization (AEMN)** technique, with a complete evaluation pipeline on the MMU Iris Database. The project explores the impact of different normalization strategies on iris recognition performance using a standard feature extraction (ResNet18) and classification (SVM) framework.

> **Note:** The system did not achieve state‑of‑the‑art performance on the MMU dataset. The results highlight the critical importance of robust segmentation and domain‑specific features in iris recognition. The code is shared as a transparent baseline and a learning resource for the community.

---

## Table of Contents
- [Aim and Objectives](#aim-and-objectives)
- [Problem Statement](#problem-statement)
- [Methodology](#methodology)
  - [Segmentation](#segmentation)
  - [Normalization Methods](#normalization-methods)
  - [Feature Extraction](#feature-extraction)
  - [Classification and Evaluation](#classification-and-evaluation)
- [Dataset](#dataset)
- [Results Summary](#results-summary)
- [Lessons Learned and Future Work](#lessons-learned-and-future-work)
- [Requirements](#requirements)
- [Usage](#usage)
- [Repository Structure](#repository-structure)
- [License](#license)
- [Citation](#citation)

---

## Aim and Objectives

**Aim:**  
To investigate the effect of adaptive elliptical normalization on iris texture preservation and recognition accuracy compared to traditional circular and uniform sampling methods.

**Objectives:**
1. Implement four normalization techniques: Daugman’s rubber‑sheet model, Circular Adaptive, Elliptic Uniform, and the proposed AEMN.
2. Design a robust ellipse detection method for pupil and iris localisation.
3. Extract discriminative features using a pre‑trained ResNet18 network.
4. Evaluate performance through identification (Rank‑1, Rank‑5, CMC), verification (EER, decidability, ROC/DET), cross‑eye matching, and statistical significance tests.
5. Analyze the strengths and weaknesses of each method and identify bottlenecks in the pipeline.

---

## Problem Statement

Iris recognition is widely considered one of the most accurate biometric modalities. However, its performance heavily depends on two critical steps: **accurate segmentation** of the iris region and **invariant feature extraction**. Many existing normalization techniques assume circular pupil and iris boundaries and sample uniformly, which may not preserve texture details in regions with high anatomical variability (e.g., crypts, furrows). Moreover, segmentation errors can propagate through the entire pipeline, rendering subsequent steps ineffective.

This project addresses the question: **Can an adaptive elliptical sampling strategy (AEMN) that varies radial density based on local edge information improve iris recognition accuracy compared to conventional methods?** The study also examines the practical challenges of implementing such a system on a publicly available dataset (MMU).

---

## Methodology

The overall pipeline consists of four main stages: segmentation, normalization, feature extraction, and classification.

### Segmentation

A custom ellipse detection algorithm is used to localise the pupil and iris boundaries:
- **Edge detection** via Canny.
- **Contour extraction** and ellipse fitting using `cv2.fitEllipse`.
- **Radius‑based filtering** to separate candidate ellipses into pupil (r < 60) and iris (r > 80).
- **Fallback** to circular Hough transform or default values if detection fails.

The detected ellipses are represented as `(cx, cy, a, b, angle)`, where `a` and `b` are semi‑axis lengths.

### Normalization Methods

All methods map the annular iris region to a fixed‑size rectangular strip of **64×360 pixels** (radial × angular). Occlusion of the top and bottom 10% simulates eyelid/eyelash interference.

1. **Daugman (Rubber‑sheet)**  
   - Forces pupil and iris boundaries to be concentric circles (average of ellipse axes).  
   - Uniform radial sampling.

2. **Circular Adaptive**  
   - Same circular boundaries as Daugman.  
   - Radial sampling density is modulated by local edge strength: more samples in high‑texture regions.

3. **Elliptic Uniform**  
   - Uses the detected elliptical boundaries directly.  
   - Uniform radial sampling.

4. **AEMN (Adaptive Elliptical Mesh Normalization)**  
   - Elliptical boundaries preserved.  
   - Radial sampling density varies along each angular column based on the average edge magnitude (computed via Sobel filter) along that column.  
   - The number of samples per column is `n = H * (1 + 0.5 * (avg_sal - sal_min)/sal_ptp)`, where `H = 64`.

### Feature Extraction

Normalised iris strips are resized to **224×224** and converted to three‑channel images to match the input of a pre‑trained **ResNet18** model (without the final classification layer). Features are extracted from the penultimate layer (512‑D vector) for each image.

### Classification and Evaluation

- **Identification:** 5‑fold cross‑validation with an SVM (RBF kernel). Metrics: accuracy, Rank‑1, Rank‑5, CMC curve, and 95% confidence intervals via bootstrap.
- **Verification:** Pairwise cosine similarity between feature vectors. Metrics: EER, decidability (d′), ROC and DET curves.
- **Cross‑eye challenge:** SVM trained on left eyes, tested on right eyes (and vice versa).
- **Ablation study:** Fine‑tuning ResNet18 on the normalized images (using the same 5‑fold split).
- **Statistical significance:** Paired t‑test on fold accuracies between methods.
- **Timing:** Average execution time per image (segmentation + normalization) for each method.

All results are saved as plots (PNG) and CSV files.

---

## Dataset

The **MMU Iris Database** is used (available from [Multimedia University](http://www.cs.princeton.edu/~andyz/irisrecognition/)). It contains:
- 100 subjects
- Left and right eye images (5 per eye, 10 total per subject)
- Image format: 320×240 pixels, BMP
- Captured under visible lighting with varying degrees of occlusion and reflection

**Segmentation statistics** (computed from our ellipse detector):

| Metric           | Mean  | Std   | Min   | Max   |
|------------------|-------|-------|-------|-------|
| Pupil radius (px)| 16.3  | 8.3   | 7.9   | 50.3  |
| Iris radius (px) | 40.9  | 29.4  | 8.1   | 131.5 |
| Pupil/Iris ratio | 0.54  | 0.27  | 0.12  | 1.42  |

The large variation and unrealistic minima (iris radius 8 px) indicate frequent segmentation failures.

---

## Results Summary

All four methods produced nearly identical, low performance:

| Method         | Accuracy | Rank‑1 | Rank‑5 | EER    | d′    |
|----------------|----------|--------|--------|--------|-------|
| daugman        | 0.091    | 0.093  | 0.276  | 0.473  | 0.102 |
| circ_adaptive  | 0.102    | 0.098  | 0.262  | 0.473  | 0.102 |
| ellip_uniform  | 0.082    | 0.089  | 0.233  | 0.478  | 0.100 |
| aemn           | 0.089    | 0.091  | 0.238  | 0.480  | 0.097 |

- **Identification accuracy** (~9‑10%) is only slightly above chance (number of classes ≈ 100 → chance ≈ 1%).
- **Rank‑5 accuracy** (~25%) still very low.
- **Verification EER** (~0.47) close to random (0.5).
- **Decidability** (~0.1) far below the typical threshold for usable systems (d′ > 3).
- **Cross‑eye accuracy** ~5% (chance level).
- **Ablation study** (fine‑tuned ResNet) improved accuracy to 13‑19%, still far from acceptable.

**Statistical significance:** Paired t‑tests revealed no significant differences between methods (p > 0.05). All methods fail similarly.

**Example visualisations** (see `results/` folder):
- Confusion matrices show near‑uniform misclassification.
- Score distributions for genuine and impostor pairs overlap completely.
- t‑SNE plot of AEMN features shows no class separation.
- Segmentation overlay images illustrate frequent mis‑localisation.

---

## Lessons Learned and Future Work

### Key Takeaways
1. **Segmentation is the critical bottleneck.** The ellipse detection method, while reasonably designed, fails on a significant portion of MMU images, producing corrupted normalised strips. Without accurate localisation, no normalization or feature extraction can succeed.
2. **Generic features are insufficient.** Pre‑trained ResNet features are optimised for natural images, not the fine texture of iris patterns. Domain‑specific features (e.g., Gabor filters, LBP) or end‑to‑end learning on iris data would likely perform better.
3. **Adaptive sampling cannot compensate for segmentation errors.** AEMN’s potential advantage remains untested because the input images are already degraded.
4. **Visible‑wavelength iris recognition is challenging.** MMU images contain reflections, off‑angle gaze, and occlusions that are hard to handle with traditional computer vision techniques.

### Future Work
- Replace the custom ellipse detector with a **deep learning‑based segmentation model** (e.g., U‑Net, IrisSeg) trained on iris datasets.
- Use **iris‑specific feature extractors** such as log‑Gabor filters or train a CNN from scratch on normalized iris strips.
- Evaluate on a cleaner near‑infrared dataset (e.g., CASIA‑IrisV1) to isolate the effect of normalization methods.
- Incorporate robust eyelid/eyelash masking using segmentation masks rather than fixed occlusions.
- Explore end‑to‑end architectures that jointly learn segmentation and recognition.

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
- openpyxl
- statsmodels
- PyTorch (torch, torchvision)

Install all dependencies with:
```bash
pip install -r requirements.txt

Citation
If you use this code or findings in your research, please cite:

Oyeniran, O. (2026). Iris Recognition with Adaptive Elliptical Mesh Normalization. 
GitHub repository:[ https://github.com/yourusername/IrisRecognition_AEMN](https://github.com/Oluwashina90/Adaptive-Elliptical-Mesh-Normalization-for-Robust-Iris-Texture-Preservation)
