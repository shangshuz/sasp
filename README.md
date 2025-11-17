# SASP Score (TGAE)

Transformer-based Graph-ish Autoencoder (TGAE) to compute a **Senescence-Associated Secretory Phenotype (SASP) score** from a panel of 38 inflammatory/protein markers. This repository includes:

- A PyTorch model (`TGAE`) for encoding protein profiles and predicting a scalar SASP score
- Utilities for data cleaning and UK Biobank distribution matching
- Helpers to compute the SASP score for new samples
- Simple fine-tuning routines and an example notebook

---

## Repo Structure

```
.
├── example.ipynb           # End-to-end demo: loading data, fine-tuning, and evaluating
├── tgae.py                 # Model: VariableEncoder/Decoder + TGAE
├── proteins_info.py        # Protein list, UKB distribution stats, data cleaning helpers
├── get_index.py            # Batch & single-row SASP score computation helpers
├── fine_tune.py            # Train/valid split and fine-tuning loop utilities
├── utils.py                # Training utilities, losses, embedding helpers, reproducibility
└── tgae_pre_trained.pth    # Pre-trained TGAE checkpoint
```

---

## Installation

Python 3.9+ recommended.

```bash
# install dependencies
pip install torch numpy pandas matplotlib
```

---

## The 38 Required Proteins

Your input must contain **exact all** of the following columns (case-insensitive accepted by the cleaner, but normalized to UPPERCASE internally):

```
ANG, CCL13, CCL2, CCL20, CCL3, CCL4, CHI3L1, CSF2, CXCL1, CXCL10, CXCL8, CXCL9, FGA, FGF2, FSTL3, GDF15, HGF, ICAM1, IGFBP2, IGFBP6, IL1B, IL4, IL5, IL6, IL6ST, LEP, MIF, MMP12, PGF, PLAUR, SERPINE1, TF, TIMP1, TNFRSF11B, TNFRSF1A, TNFRSF1B, TNFSF10, VEGFA
```

---

## Quickstart: Compute SASP Score for New Data

> See `example.ipynb` for a full, runnable notebook that loads CSVs, fine-tunes, and evaluates correlations (e.g., with age).

---

## API Overview

### `tgae.TGAE`
- **`forward(x, var_label)`** → `(latent, recon)`  
  - `x`: `(batch, P)` protein values  
  - `var_label`: `(1, P)` integer embedding of variable names  
  - Returns a **latent scalar prediction** (the SASP score) and a **reconstruction** of inputs.
- **`predict(x, var_label)`** → `latent`  
  - Convenience method to get only the SASP score.

### `proteins_info.py`
- `protein_list`: canonical ordered tuple of the 38 proteins.  
- `match_ukb_dist(df)`: standardizes each protein column of `df` to match UK Biobank means/SDs.  
- `clean_data(df, tunning=False, proteins_list=protein_list)`: validates presence of required proteins (case-insensitive); optionally finds the `age` column when `tunning=True`.

### `get_index.py`
- `gen_single_index(row, tgae, device)`: compute SASP score for a single row.  
- `gen_index(df, tgae, device)`: compute SASP score for all rows (returns `list[float]`).

### `utils.py`
- Embedding helpers to map protein names to integer IDs for the transformer.
- Reproducibility helpers, data preparation, loss routines, and a simple training loop.

### `fine_tune.py`
- `split_df(n_rows, train_size, valid_size)` → train/valid/test indices  
- `train(...)` → fine-tuning loop wrapper

---

## Tips & Gotchas

- **Column names**: Use the exact protein names above. `clean_data` will raise if names are missing or duplicated in a case-insensitive way.
- **Standardization**: You normally don’t need to standardize beforehand—the helper aligns to UKB distribution—but ensure your input scale is reasonable (floats).
- **Missing values**: For inference, missing per-row values are dropped for that row. For training, ensure consistent preprocessing.
- **Determinism**: Set seeds to make splits and training reproducible (see `utils`).

---

## Example Notebook

Open `example.ipynb` for a complete walkthrough:
- Set hyperparameters
- Load & clean data
- Fine-tune the model
- Compute and inspect SASP score

---

## Citation / License

If you use this code in academic work, please cite appropriately.  
**License**: _Fill in your preferred license here (e.g., MIT, Apache-2.0)._


---

## Acknowledgements

Thanks to UK Biobank–based statistics included in `proteins_info.py` for distribution matching.

