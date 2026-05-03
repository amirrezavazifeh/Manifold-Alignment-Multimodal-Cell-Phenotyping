<div align="center">

<h1>Manifold Alignment for Label-Free Cell Phenotyping in Multimodal Microscopy
</h1>

<p>
  <a href="https://www.optica.org/events/congress/biophotonics_congress/"><img src="https://img.shields.io/badge/Conferene-Optica_Biophotonics_Congress_2026-orange?style=flat-square" alt="Optica Biophotonics Congress 2026"></a>
</p>

<p>
  <a href="https://amirrezavazifeh.github.io/">Amir Reza Vazifeh</a><sup>1</sup> &nbsp;·&nbsp;
  <a href="https://ece.princeton.edu/people/jason-w-fleischer">Jason W. Fleischer</a><sup>1</sup> *</sup>
</p>

<p>
  <sup>1</sup> Department of Electrical and Computer Engineering, Princeton University, Princeton, New Jersey 08544, USA<br>
</p>

<p><em>Optica Biophotonics Congress</em>, 2026</p>

</div>

---

## Overview

---

## Method


---

## Result

---

## Repository Structure

---

## Contact

For questions or issues, please contact [amir.vazifeh@princeton.edu].

---

## Citation



# UMAP vs MANE: Dimensionality Reduction for Single and Multi-Modal Data

> **UMAP** reduces a single high-dimensional dataset to a low-dimensional embedding. **MANE** extends this to *multiple* datasets, enforcing a shared embedding across modalities.

---

## Table of Contents

- [Overview](#overview)
- [Problem Setup](#problem-setup)
- [Algorithm Comparison](#algorithm-comparison)
  - [Step 1: High-Dimensional Graph](#step-1-high-dimensional-graph)
  - [Step 2: Low-Dimensional Graph](#step-2-low-dimensional-graph)
  - [Step 3: Loss Optimization](#step-3-loss-optimization)
- [MANE Loss Function (Full Form)](#mane-loss-function-full-form)
- [Example Application: Brightfield & Phase-Contrast Cell Imaging](#example-application-brightfield--phase-contrast-cell-imaging)
- [References](#references)

---

## Overview

| Property | UMAP | MANE |
|---|---|---|
| **Inputs** | 1 high-D dataset | $m \geq 2$ high-D datasets |
| **Output** | 1 low-D embedding | 1 **shared** low-D embedding |
| **Constraint** | None | $y_i^{(1)} = y_i^{(2)} = \cdots = y_i^{(m)}$ |
| **Use case** | Single-modality DR | Multi-modal integration & alignment |

---

## Problem Setup

### UMAP

$$X = \{x_i \in \mathbb{R}^n \mid i = 1, \dots, N\}, \qquad Y = \{y_i \in \mathbb{R}^d \mid i = 1, \dots, N\}, \quad d \ll n$$

### MANE

$$X^{(r)} = \{x_i^{(r)} \in \mathbb{R}^n \mid i = 1, \dots, N\}, \quad r = 1, \dots, m$$

$$Y = \{y_i \in \mathbb{R}^d \mid i = 1, \dots, N\}, \quad d \ll n$$

MANE takes $m$ high-dimensional datasets $X^{(1)}, \dots, X^{(m)}$ — each with $N$ matched samples — and finds a **single** low-dimensional space $Y$ that faithfully represents the manifold structure of all modalities simultaneously.

---

## Algorithm Comparison

### Step 1: High-Dimensional Graph

Both methods construct a weighted graph that captures pairwise structure in the original high-dimensional space.

**UMAP:**
$$p_{ij} = f_H\!\left(d\!\left(x_i,\, x_j\right)\right)$$

**MANE** (one graph per modality):
$$p_{ij}^{(r)} = f_H\!\left(d_H\!\left(x_i^{(r)},\, x_j^{(r)}\right)\right), \quad r = 1, \dots, m$$

Here $d_H(\cdot, \cdot)$ is the distance function in high-dimensional space and $f_H$ is a weighting function (e.g., based on $k$-NN with adaptive bandwidths, as in UMAP).

---

### Step 2: Low-Dimensional Graph

A second weighted graph is constructed in the target low-dimensional space, used to compare against the high-dimensional structure.

**UMAP:**
$$q_{ij} = f_L\!\left(d\!\left(y_i,\, y_j\right)\right)$$

**MANE** (one graph per modality, but all using the **same** $y_i$):
$$q_{ij}^{(r)} = f_L\!\left(d_L\!\left(y_i^{(r)},\, y_j^{(r)}\right)\right), \quad r = 1, \dots, m$$

The key constraint is that $y_i^{(r)}$ is **shared** across all modalities: $y_i^{(1)} = y_i^{(2)} = \cdots = y_i^{(m)}$. Both $d_H$, $d_L$ and $f_H$, $f_L$ are selected identically to UMAP.

---

### Step 3: Loss Optimization

**UMAP** minimises a cross-entropy loss between the high- and low-dimensional graphs:

$$l = \sum_{(i,j)} l\!\left(p_{ij},\, q_{ij}\right)$$

**MANE** minimises the **sum** of per-modality losses subject to the shared-embedding constraint:

$$l = \sum_{r=1}^{m} \sum_{(i,j)} l\!\left(p_{ij}^{(r)},\, q_{ij}^{(r)}\right) \quad \text{s.t.} \quad y_i^{(1)} = y_i^{(2)} = \cdots = y_i^{(m)}$$

---

## MANE Loss Function (Full Form)

For the two-modality case ($m = 2$), the individual loss term is the binary cross-entropy:

$$l\!\left(p_{ij}^{(r)},\, q_{ij}^{(r)}\right) = p_{ij}^{(r)} \log \frac{p_{ij}^{(r)}}{q_{ij}^{(r)}} + \left(1 - p_{ij}^{(r)}\right) \log \frac{1 - p_{ij}^{(r)}}{1 - q_{ij}^{(r)}}$$

So the full MANE objective is:

$$l = \sum_{r=1}^{2} \sum_{(i,j)} \left( p_{ij}^{(r)} \log \frac{p_{ij}^{(r)}}{q_{ij}^{(r)}} + \left(1 - p_{ij}^{(r)}\right) \log \frac{1 - p_{ij}^{(r)}}{1 - q_{ij}^{(r)}} \right) \quad \text{s.t.} \quad y_i^{(1)} = y_i^{(2)}$$

The constraint $y_i^{(1)} = y_i^{(2)}$ forces both modalities to produce the **same** low-dimensional coordinate for each sample $i$, enabling integration of complementary information while preserving the manifold structure of both datasets.

---

## Example Application: Brightfield & Phase-Contrast Cell Imaging

As a concrete use-case, we acquired two imaging modalities of CHO (Chinese Hamster Ovary) cells:

- **Modality 1** $X^{(1)}$: Brightfield RGB images at multiple focal planes
- **Modality 2** $X^{(2)}$: Phase-contrast RGB images at multiple focal planes

Individual cells were segmented using [Cellpose](https://github.com/MouseLand/cellpose) and resized to $64 \times 64$ pixels, yielding feature vectors of dimension:

$$n = 64 \times 64 \times 3 = 12{,}288$$

Each cell is thus represented as:

$$x_i^{(1)},\; x_i^{(2)} \in \mathbb{R}^{12288}$$

MANE is then applied to find a shared low-dimensional embedding $y_i \in \mathbb{R}^d$ ($d \ll 12{,}288$) that simultaneously captures the structure from both imaging modalities, enabling integrated visualisation and downstream analysis of cell morphology.

> **Note:** A naive baseline would concatenate both modalities into a single $24{,}576$-dimensional vector and apply UMAP. MANE instead preserves the independence of each modality's graph structure while still enforcing alignment, which can yield better-separated clusters when the two modalities carry complementary signals.

---

## References

- McInnes, L., Healy, J., & Melville, J. (2018). **UMAP: Uniform Manifold Approximation and Projection for Dimension Reduction**. *arXiv:1802.03426*. [Paper](https://arxiv.org/abs/1802.03426)

- Islam, M. T., et al. (2022). **Manifold-aligned Neighbor Embedding**. *ICLR 2022 Workshop on Geometrical and Topological Representation Learning*. [Paper](https://openreview.net/)

- Stringer, C., Wang, T., Michaelos, M., & Pachitariu, M. (2021). **Cellpose: a generalist algorithm for cellular segmentation**. *Nature Methods*, 18(1), 100–106. [Paper](https://doi.org/10.1038/s41592-020-01018-x)
