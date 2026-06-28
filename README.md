# TSItransRL

A reinforcement-learning framework for **de novo molecular generation** using sequence-based transformers.

> **Repository:** `leicaohmu/TSItransRL`  
> **Primary language:** Python

---

## Overview

TSItransRL combines a transformer-style sequence model with reinforcement learning to generate molecules (typically represented as SMILES) that optimize user-defined objectives (e.g., drug-likeness, synthetic accessibility, target-related properties).

A typical workflow is:

1. Prepare and clean molecular data.
2. Build tokenization / vocabulary assets.
3. Train or load a prior (supervised) sequence model.
4. Fine-tune with reinforcement learning using a reward function.
5. Generate candidate molecules.
6. Evaluate, filter, and export results.

---

## Workflow at a Glance

```text
Raw molecular data (SMILES/CSV)
        │
        ▼
Data cleaning + canonicalization + split
        │
        ▼
Tokenization / vocabulary construction
        │
        ▼
Supervised pretraining (prior model)
        │
        ▼
RL fine-tuning with custom reward(s)
        │
        ▼
Molecule sampling / generation
        │
        ▼
Post-processing + scoring + export
```

---

## Software Dependencies

The project is Python-based. Install the dependencies used by your scripts and training pipeline.

### Core dependencies (typical)

- Python 3.9+ (recommended)
- PyTorch
- NumPy
- pandas
- RDKit
- scikit-learn
- tqdm
- PyYAML / OmegaConf (if configs are YAML-driven)

### Optional dependencies

- CUDA-enabled PyTorch (for GPU training)
- JupyterLab/Notebook (for interactive experiments)
- matplotlib / seaborn (for analysis plots)

> If this repository includes a `requirements.txt`, `environment.yml`, or `pyproject.toml`, those files should be treated as the source of truth.

---

## Installation

### 1) Clone repository

```bash
git clone https://github.com/leicaohmu/TSItransRL.git
cd TSItransRL
```

### 2) Create a virtual environment

#### Using `venv`

```bash
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows PowerShell
```

#### (Optional) Using conda

```bash
conda create -n tsitransrl python=3.10 -y
conda activate tsitransrl
```

### 3) Install dependencies

Pick one that matches repository files:

```bash
pip install -r requirements.txt
```

or

```bash
conda env update -f environment.yml
```

or

```bash
pip install -e .
```

### 4) Verify install

```bash
python -c "import torch; print('torch ok')"
python -c "import rdkit; print('rdkit ok')"
```

---

## Project Structure (Suggested Interpretation)

> Exact paths may vary; adjust commands to match this repository’s scripts/modules.

```text
TSItransRL/
├── data/                 # raw + processed datasets
├── configs/              # training/RL/generation configs
├── models/               # model definitions / checkpoints
├── scripts/              # executable scripts
├── outputs/              # generated molecules and logs
└── README.md
```

---

## Data Preparation

Molecular generation quality strongly depends on data quality.

### Input format

- CSV/TXT with one SMILES per row/line (common patterns):
  - `smiles`
  - `SMILES`
  - first column only

### Recommended preparation steps

1. **Load data** and keep valid, non-empty SMILES.
2. **Canonicalize** with RDKit.
3. **Remove duplicates**.
4. **Filter by length / atom constraints** (optional).
5. **Train/valid/test split** (e.g., 80/10/10).
6. **Build vocabulary/tokenizer** from training split.
7. **Persist artifacts** (`vocab.json`, processed CSV, split files).

### Example (generic)

```bash
python scripts/prepare_data.py \
  --input data/raw/molecules.csv \
  --smiles-column smiles \
  --output-dir data/processed \
  --build-vocab \
  --train-ratio 0.8 --valid-ratio 0.1 --test-ratio 0.1
```

**Outputs typically include:**

- `data/processed/train.csv`
- `data/processed/valid.csv`
- `data/processed/test.csv`
- `data/processed/vocab.json`

---

## Model Training

TSItransRL commonly involves two training phases:

1. **Prior (supervised) training** to learn valid molecular syntax/patterns.
2. **Reinforcement learning fine-tuning** to optimize target objectives.

### A) Train prior model

```bash
python scripts/train_prior.py \
  --config configs/prior.yaml \
  --train data/processed/train.csv \
  --valid data/processed/valid.csv \
  --vocab data/processed/vocab.json \
  --save-dir outputs/prior
```

Common configuration knobs:

- model size (`d_model`, layers, heads)
- sequence length
- batch size
- learning rate / scheduler
- dropout
- number of epochs

### B) RL fine-tuning

```bash
python scripts/train_rl.py \
  --config configs/rl.yaml \
  --prior-checkpoint outputs/prior/best.pt \
  --save-dir outputs/rl
```

Common RL knobs:

- RL algorithm settings (policy gradient / variants)
- reward coefficients / multi-objective weights
- KL regularization to prior
- rollout batch size
- update frequency / steps

---

## Molecule Generation

After training, sample molecules from the fine-tuned policy.

```bash
python scripts/generate.py \
  --checkpoint outputs/rl/best.pt \
  --num-samples 10000 \
  --max-length 120 \
  --output outputs/generated_molecules.csv
```

Recommended post-generation analyses:

- **Validity**: fraction of chemically valid molecules
- **Uniqueness**: proportion of non-duplicate molecules
- **Novelty**: not present in training set
- **Property distributions**: QED, logP, SA, MW, etc.
- **Top-k selection** by reward / objective score

---

## Reward Design Notes

A robust reward function is critical for RL-based molecular generation.

Typical reward components:

- Drug-likeness (QED)
- Synthetic accessibility (SA)
- Physicochemical constraints (MW, logP, TPSA)
- Substructure constraints / medicinal chemistry filters
- Activity proxy (if a predictive model is available)

Practical tips:

- Normalize component scores before weighting.
- Start with simple objectives, then add complexity.
- Monitor mode collapse (low diversity).
- Track validity/uniqueness alongside reward.

---

## Reproducibility

For stable and repeatable experiments:

- Set random seeds (`python`, `numpy`, `torch`).
- Log full config files and git commit hash.
- Save tokenizer/vocab with each run.
- Store checkpoints and generated samples per run.
- Version data preprocessing outputs.

---

## Troubleshooting

### RDKit import issues

- Ensure the correct Python environment is active.
- Prefer conda installation for RDKit compatibility if needed.

### CUDA / GPU not used

- Confirm CUDA-compatible PyTorch build.
- Check `torch.cuda.is_available()`.

### Low validity in generated molecules

- Increase prior training quality.
- Tighten tokenization/vocab consistency.
- Add stronger KL regularization during RL.
- Reduce aggressive reward shaping.

### Training instability

- Lower learning rate.
- Increase batch size (if memory allows).
- Clip gradients.
- Monitor reward scale and outliers.

---

## Citation

If you use this project in research, please cite the associated paper/repository.

```bibtex
@misc{tsitransrl,
  title={TSItransRL},
  author={Project Authors},
  year={2026},
  howpublished={\url{https://github.com/leicaohmu/TSItransRL}}
}
```

---

## License

Please refer to the repository’s `LICENSE` file for terms of use.
