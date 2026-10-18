# Human Connectome Project (HCP): Cortical Morphology & Hemispheric Laterality

Analysis notebooks, data references, and a hands-on workshop for exploring **cortical structure** (thickness, curvature, sulcal depth) and **hemispheric laterality** using the [Human Connectome Project (HCP)](https://www.humanconnectome.org/) S1200 release.

![Python](https://img.shields.io/badge/python-3.9%2B-blue)
![Data](https://img.shields.io/badge/data-HCP%20S1200-purple)
![Status](https://img.shields.io/badge/status-active-brightgreen)

---

## Overview

The HCP S1200 release provides high-quality structural MRI from ~1,200 healthy young adults, processed with the HCP minimal preprocessing pipelines and FreeSurfer. This repository collects the analyses built on those structural measures and explains how to reproduce them.

**What you can do here**

- Explore the **S1200 data dictionary** to find the variables you need
- Examine **cortical thickness**, **cortical curvature**, and **sulcal depth** across parcels
- Compute **cortical thickness correlation** (structural covariance) between regions
- Quantify **left–right laterality** in the included **laterality workshop**

---

## Repository structure

```
.
├── README.md
├── data/
│   ├── cortical_curvature/        # Curvature measures per parcel/subject
│   ├── cortical_thickness/        # Thickness measures per parcel/subject
│   ├── sulcal_depth/              # Sulcal depth measures per parcel/subject
│   ├── cortical_thickness_correlation/   # Region-by-region correlation matrices
│   └── S1200_data_dictionary/     # Variable definitions for the S1200 release
├── workshop/
│   └── laterality_workshop/       # Step-by-step laterality analysis
├── requirements.txt
└── LICENSE
```

> Adjust folder names above to match your actual layout.

| Component | Description |
|---|---|
| **Cortical thickness** | Distance between the white-matter and pial surfaces (mm) |
| **Cortical curvature** | Local folding geometry of the cortical surface |
| **Sulcal depth** | Depth of cortical folds relative to the mid-surface |
| **Cortical thickness correlation** | Inter-regional covariance of thickness across subjects |
| **S1200 data dictionary** | Column-by-column definitions of the HCP behavioral/imaging variables |
| **Laterality workshop** | Tutorial on computing and testing hemispheric asymmetry |

---

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/FarimaM/<repository-name>.git
cd <repository-name>
```

### 2. Set up the environment

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Suggested dependencies: `numpy`, `pandas`, `scipy`, `matplotlib`, `seaborn`, `nibabel`, `jupyter`.

### 3. Get the HCP data

Raw HCP data **cannot be redistributed** and must be obtained directly from the source:

1. Create an account at [ConnectomeDB](https://db.humanconnectome.org/).
2. Accept the **Open Access Data Use Terms**. For restricted variables (e.g., family structure, exact age), also apply for **Restricted Access**.
3. Download the S1200 release (structural / FreeSurfer outputs and the unrestricted/restricted CSVs).
4. Place the files in `data/` following the structure above.

---

## Quick start

### Load thickness data and compute structural covariance

```python
import pandas as pd

thickness = pd.read_csv("data/cortical_thickness/thickness.csv", index_col=0)

# Region-by-region correlation across subjects
corr = thickness.corr(method="pearson")
corr.to_csv("data/cortical_thickness_correlation/thickness_corr.csv")
```

### Compute a laterality index (LI)

For each region, the laterality index compares left and right hemisphere values:

$$
LI = \frac{L - R}{L + R}
$$

- `LI > 0`: leftward asymmetry
- `LI < 0`: rightward asymmetry

```python
import pandas as pd
from scipy import stats

lh = pd.read_csv("data/cortical_thickness/lh_thickness.csv", index_col=0)
rh = pd.read_csv("data/cortical_thickness/rh_thickness.csv", index_col=0)

li = (lh - rh) / (lh + rh)

# One-sample t-test per region against zero asymmetry
t, p = stats.ttest_1samp(li, 0, axis=0)
results = pd.DataFrame({"t": t, "p": p}, index=li.columns)
print(results.sort_values("p").head())
```

Remember to correct for multiple comparisons (FDR or Bonferroni) across regions.

---

## Laterality workshop

The `workshop/laterality_workshop` folder is a guided tutorial that covers:

1. Loading left/right hemisphere measures
2. Computing laterality indices
3. Statistical testing and multiple-comparison correction
4. Visualizing asymmetry across the cortex
5. Relating asymmetry to behavioral variables from the S1200 data dictionary

Launch it with:

```bash
jupyter notebook workshop/laterality_workshop/
```

---

## Reproducibility notes

- Use the **same parcellation** (e.g., Desikan–Killiany or Glasser/HCP-MMP1.0) across all measures.
- Consider covariates such as **age, sex, and total intracranial volume** when comparing thickness.
- HCP includes **related individuals (twins and siblings)**. Account for family structure (e.g., permutation tests respecting family blocks, or mixed models) to avoid inflated statistics.
- Fix random seeds for any permutation or bootstrap procedure.

---

## Data use and citation

This repository must not contain subject-level restricted data. Share only derived or aggregate results permitted under the HCP Data Use Terms.

If you use HCP data, please include the standard acknowledgment:

> Data were provided by the Human Connectome Project, WU-Minn Consortium (Principal Investigators: David Van Essen and Kamil Ugurbil; 1U54MH091657) funded by the 16 NIH Institutes and Centers that support the NIH Blueprint for Neuroscience Research; and by the McDonnell Center for Systems Neuroscience at Washington University.

And cite:

> Van Essen, D. C., et al. (2013). The WU-Minn Human Connectome Project: An overview. *NeuroImage*, 80, 62–79.

If you use this repository, please cite it:

```bibtex
@misc{FarimaM_hcp,
  author = {Farima},
  title  = {Human Connectome Project: Cortical Morphology and Laterality},
  year   = {2026},
  url    = {https://github.com/FarimaM/<repository-name>}
}
```

---

## Roadmap

- [ ] Add environment file (`environment.yml`) for one-command setup
- [ ] Add surface-based visualizations of thickness and asymmetry
- [ ] Add brain–behavior association analyses
- [ ] Add automated tests and a CI workflow

---

## Contributing

Issues and pull requests are welcome. Please open an issue first to discuss major changes.

## License

Code is released under the [MIT License](LICENSE). HCP data are governed separately by the HCP Data Use Terms.

## Contact

**Farima**, Computational Neuroscience Researcher
GitHub: [@FarimaM](https://github.com/FarimaM)
