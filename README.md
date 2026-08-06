# MediSum — Medical Report Summarization using LLMs

DA1 mini project: LLM-based discharge summary generation with a fact-verification/grounding layer.

## Dataset
MIMIC-IV-Ext-BHC v1.2.0 (PhysioNet, DOI 10.13026/5gte-bv70).
This is credentialed patient data and is **not included in this repo**.

Each contributor must:
1. Register and get credentialed on PhysioNet (CITI training + DUA)
2. Request access: https://physionet.org/content/labelled-notes-hospital-course/1.2.0/
3. Download `mimic-iv-bhc.csv` (2.5 GB) and place it in `data/raw/`

## Structure
- `data/` — raw/processed data (gitignored, not tracked)
- `scripts/` — data download & preprocessing
- `src/` — pipeline code (generation, verification)
- `notebooks/` — exploration
