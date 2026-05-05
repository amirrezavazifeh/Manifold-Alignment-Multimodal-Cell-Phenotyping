<div align="center">
<h1>Manifold Alignment for Label-Free Cell Phenotyping in Multimodal Microscopy</h1>
<p>
  <a href="https://www.optica.org/events/congress/biophotonics_congress/"><img src="https://img.shields.io/badge/Conference-Optica_Biophotonics_Congress_2026-orange?style=flat-square" alt="Optica Biophotonics Congress 2026"></a>
</p>
<p>
  <a href="https://amirrezavazifeh.github.io/">Amir Reza Vazifeh</a><sup>1</sup> &nbsp;·&nbsp;
  <a href="https://ece.princeton.edu/people/jason-w-fleischer">Jason W. Fleischer</a><sup>1,*</sup>
</p>
<p>
  <sup>1</sup> Department of Electrical and Computer Engineering, Princeton University, Princeton, NJ 08544, USA<br>
  <sup>*</sup> jasonf@princeton.edu
</p>
<p><em>Optica Biophotonics Congress</em>, 2026</p>
</div>

---

## Overview

Cell viability assessment — determining the ratio of live to dead cells — is fundamental to biological research, drug development, and clinical diagnostics. Conventional methods require adding fluorescent dyes (e.g., Acridine Orange / Propidium Iodide) to stain cells before imaging. Beyond the labor-intensive sample preparation, staining can damage or kill cells, some cells appear in both fluorescent channels, and images are susceptible to photobleaching.

**Can unstained brightfield or phase-contrast images contain sufficient information for viability analysis — without any fluorescent labels?**

This work shows the answer is yes, and introduces a multimodal unsupervised framework to achieve it.

---

## Motivation

### Approach 1: Virtual Staining
One natural idea is to train a deep neural network (e.g., a U-Net) to predict fluorescent images directly from transmitted-light images. While promising, this approach has critical limitations:
- Requires large, carefully registered image pairs for supervised training.
- Some cells appear in both fluorescent channels, creating ambiguous labels.
- Models do not generalize well across different sample types, imaging modalities, or focus conditions.

### Approach 2: Cell Segmentation + Classification
A more principled pipeline first segments individual cells (e.g., using Cellpose or Faster R-CNN), then trains a classifier (CNN) to predict live vs. dead. This avoids full image-to-image mapping and directly targets viability counting. However, supervised classification still has failure modes:
- Intra-class variation among dead cells can confuse trained models.
- Inaccurate bounding boxes lead to outlier predictions.
- Most critically, **no labeled dataset exists for unstained samples** — dimensionality reduction plots show that stained and unstained brightfield images live on different manifolds, meaning a supervised model trained on stained data will not reliably generalize to unstained data.

### Approach 3: Unsupervised Dimensionality Reduction
Because of the domain gap between stained and unstained images, an unsupervised approach is more appropriate. Nonlinear dimensionality reduction methods such as UMAP and t-SNE project high-dimensional cell images into a 2D latent space, where live and dead cells naturally form separable clusters — with no labels required. This also enables discovery of sub-populations (e.g., two morphologically distinct types of dead cells) and outlier detection that supervised methods miss.

### Approach 4: Multimodal Integration with MANE
Standard UMAP operates on a single modality and must be applied separately to each imaging configuration. Different modalities — brightfield, phase-contrast, different focal planes — each capture complementary cellular information. **Manifold-Aligned Neighbor Embedding (MANE)** extends UMAP to jointly embed multiple modalities into a shared low-dimensional space, enforcing that the same cell maps to the same point regardless of which modality it comes from. This multimodal integration yields richer, more discriminative representations than any single modality alone.

---

## Method

MANE generalizes UMAP to multiple datasets by optimizing a joint cross-entropy loss across all modalities, subject to the constraint that each cell shares a single low-dimensional embedding across modalities:

$$\ell = \sum_{r=1}^{m} \sum_{(i,j)} \ell\!\left(p_{ij}^{(r)}, q_{ij}^{(r)}\right) \quad \text{s.t.} \quad y_i^{(1)} = y_i^{(2)} = \cdots = y_i^{(m)}$$

where $p_{ij}^{(r)}$ and $q_{ij}^{(r)}$ are the high- and low-dimensional pairwise affinities for modality $r$, constructed exactly as in UMAP. The shared embedding constraint forces the model to reconcile complementary information across modalities rather than embedding them independently.

In our experiments, brightfield and phase-contrast RGB images of CHO cells were acquired at multiple focal planes. Individual cells were segmented using Cellpose and resized to 64×64 pixels, yielding a 12,288-dimensional feature vector per cell per modality. MANE was then applied to produce a joint 2D embedding.

---

## Results

<p align="center">
  <img src="UMAP vs MANE 1.png" width="1000">
</p>

When UMAP is applied independently to brightfield and phase-contrast images, both produce two rough clusters but cannot integrate cross-modal information. A naive joint UMAP (concatenation) improves separation slightly. **MANE achieves consistently higher KNN classification accuracy across all values of k**, and produces two clean, well-separated clusters corresponding to live and dead cells. Inspecting representative samples confirms that cluster 1 contains cells with sharp, well-defined membranes (live), while cluster 2 contains cells with diffuse, fuzzy boundaries (dead).

---

## Repository Structure

```
├── data/               # Cell image patches (brightfield + phase)
├── src/
│   ├── segmentation/   # Cellpose-based cell segmentation
│   ├── embedding/      # UMAP and MANE dimensionality reduction
│   └── evaluation/     # KNN classification and visualization
├── figures/            # Output embeddings and sample images
└── README.md
```

---

## Contact

For questions or issues, please contact [amir.vazifeh@princeton.edu](mailto:amir.vazifeh@princeton.edu).

---

## Citation

```bibtex
@inproceedings{vazifeh2026manifold,
  title     = {Manifold Alignment for Label-Free Cell Phenotyping in Multimodal Microscopy},
  author    = {Vazifeh, Amir Reza and Fleischer, Jason W.},
  booktitle = {Optica Biophotonics Congress},
  year      = {2026}
}
```
