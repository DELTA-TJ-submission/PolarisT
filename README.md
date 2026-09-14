<p align="center">
  <a href="https://github.com/yh-wang1116/PolarisT/blob/main/PolarisT_icon.png">
    <img width="150" alt="PolarisT" src="./PolarisT_icon.png" />
  </a>
</p>

<h1 align="center">
  Navigating T cell transcriptomic reprogramming by an<br>
  AI virtual T-cell model PolarisT
</h1>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.0.0-62518C" alt="Version 1.0.0" />
  <img src="https://img.shields.io/badge/Python-3.10%20%7C%203.11%20%7C%203.12-3776AB?logo=python&logoColor=white" alt="Python 3.10, 3.11 or 3.12" />
</p>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Installation](#-installation)
- [Usage](#-usage)
  - [Atlas-profiled perturbation ranking](#atlas-profiled-perturbation-ranking)
  - [Atlas-unprofiled gene ranking](#atlas-unprofiled-gene-ranking)
  - [Custom phenotype gene sets](#custom-phenotype-gene-sets)
  - [Perturbation data integration](#perturbation-data-integration)
- [Tutorial](#-tutorial)
- [Repository Structure](#-repository-structure)
- [Citation](#-citation)

---

## 💡 Overview

PolarisT is a perturbation-centric AI virtual T-cell model for navigating CD8⁺ T cell transcriptomic reprogramming.

PolarisT provides a framework for learning from immune perturbation atlases, navigating among their genetic interventions and extrapolating beyond their gene coverage, linking perturbation atlases to experimentally testable strategies for CD8⁺ T cell reprogramming.

It contains three related components:

- **Perturbation data integration:** integrates large-scale single-cell perturbation data with a batch-conditioned variational autoencoder and metric-learning supervision.
- **Atlas-profiled perturbations (seen):** ranks perturbations measured in the perturbation atlas according to a user-defined T cell phenotype.
- **Atlas-unprofiled perturbations (unseen):** uses a pretrained graph-informed model to rank candidate genes that were not directly profiled in the atlas.

---

## 📦 Installation

PolarisT requires Python 3.10, 3.11 or 3.12. Create a conda environment and install the package as follows:

```bash
conda create -n polarist python=3.10
conda activate polarist
git clone https://github.com/DELTA-TJ-submission/PolarisT.git
cd PolarisT
pip install .
```

To install the optional tutorial and development dependencies:

```bash
pip install ".[demo,dev]"
```

The package includes the pretrained models and compact inference resources required by the released seen and unseen workflows. No large AnnData input is required for the bundled tutorial notebooks.

---

## 🚀 Usage

### Atlas-profiled perturbation ranking

`rank_seen_drivers()` defines a phenotype from positive and negative gene signatures, selects the corresponding extreme cells, and ranks perturbations already represented in the atlas.

```python
from polarist import rank_seen_drivers

result = rank_seen_drivers(
    positive_genes=[
        "TCF7", "LEF1", "SLAMF6", "SELL", "BCL2",
        "BCL6", "CXCR5", "CCNE1", "CCNE2",
    ],
    negative_genes=["TOX", "HAVCR2", "ENTPD1", "CD101", "CD244"],
    phenotype_name="stemness",
    extreme_fraction=0.05,
    refinement_weight=0.9,
    tf_only=True,
)

ranking = result.ranking
print(ranking.head())
```

### Atlas-unprofiled gene ranking

`rank_unseen_drivers()` ranks genes that were not directly profiled as perturbations in the released atlas.

```python
from polarist import rank_unseen_drivers

result = rank_unseen_drivers(
    positive_genes=[
        "TCF7", "LEF1", "SLAMF6", "SELL", "BCL2",
        "BCL6", "CXCR5", "CCNE1", "CCNE2",
    ],
    negative_genes=["TOX", "HAVCR2", "ENTPD1", "CD101", "CD244"],
    phenotype_name="stemness",
    extreme_fraction=0.05,
    n_genes=1000,
    tf_only=True,
)

ranking = result.ranking
print(ranking.head())
```
### Custom phenotype gene sets

PolarisT is trained for CD8⁺ T cells. Users can define custom phenotypes by providing positive and negative gene sets. For a custom gene signature, PolarisT recomputes phenotype-associated cells from the CD8⁺ T-cell expression matrix before ranking atlas-profiled perturbations or atlas-unprofiled genes.

Custom phenotype analysis requires an additional AnnData file, available from the associated Figshare record:

[Download the CD8⁺ T-cell AnnData dataset](https://doi.org/10.6084/m9.figshare.32934569)

Download `Anndata_cd8_raw.h5ad` and place it in a local data directory, for example:

```text
/path/to/polarist_data/Anndata_cd8_raw.h5ad
```

Pass the directory containing this file through `resource_dir`:

```python
from polarist import rank_seen_drivers

result = rank_seen_drivers(
    positive_genes=["NKG7", "GZMB", "IFNG"],
    negative_genes=["TOX", "PDCD1"],
    phenotype_name="cytotoxicity",
    extreme_fraction=0.05,
    refinement_weight=0.9,
    tf_only=True,
    resource_dir="/path/to/polarist_data",
)

ranking = result.ranking
print(ranking.head())
```

The same `resource_dir` argument can be used with `rank_unseen_drivers()`.

The AnnData object must contain the following fields:

```python
adata.layers["logNor"]
adata.obs["Unique_cellid"]
adata.obs["Dataset"]
adata.obs["Immune_type"]
adata.var_names
```

**Note:** `resource_dir` must point to the directory containing `Anndata_cd8_raw.h5ad`, not to the file itself.

### Perturbation data integration

The integration component uses a batch-conditioned variational autoencoder with metric-learning supervision to generate a low-dimensional representation of large-scale single-cell perturbation data. The implementation is provided in `Perturb_data_integrate/Integrate_data.py`.

The script requires a compatible AnnData object containing raw counts and the metadata fields `Dataset` and `Immune_type`. Because the input data are large, the complete AnnData object is not included. This analysis requires substantial memory and is best run with a CUDA-enabled GPU.

See [`Perturb_data_integrate/README.md`](Perturb_data_integrate/README.md) for the input requirements and command-line usage.

---

## 📖 Tutorial

The [`Tutorial/`](Tutorial/) directory contains runnable notebooks for both ranking workflows:

- [Atlas-profiled perturbation ranking](Tutorial/atlas_profiled_driver_demo.ipynb)
- [Atlas-unprofiled gene ranking](Tutorial/atlas_unprofiled_driver_demo.ipynb)

The accompanying CSV files contain example ranking outputs.

---

## 📂 Repository Structure

```text
src/polarist/
├── __init__.py       # Public package API and version
├── phenotype.py      # Gene-signature scoring and phenotype definition
├── seen.py           # Atlas-profiled perturbation ranking
├── unseen.py         # Atlas-unprofiled gene ranking
├── workflow.py       # High-level seen workflow
├── models.py         # Neural-network components for unseen inference
└── resources/        # Bundled embeddings, graph and pretrained weights

Perturb_data_integrate/
└── Integrate_data.py # Large-scale integration and metric-learning analysis

Tutorial/            # Example notebooks and ranking outputs
tests/               # Package tests
```

---

## 📝 Citation

If you find PolarisT useful for your research, please consider citing our paper:

```bibtex
@article{polarist_submitted,
  title   = {Navigating T cell transcriptomic reprogramming by an AI virtual T-cell model PolarisT},
  journal = {Submitted},
  year    = {2026}
}
```

The citation will be updated with the preprint or publication DOI once available.
