# Rule-Based ABSA Knowledge Graph for PMA Graduate Profiles

A custom NLP data pipeline built on Philippine Military Academy (PMA) Class of 1980 yearbook records. Each cadet's profile — a peer-authored description and a self-chosen quote — is treated as a dual window into external perception and internal self-expression. These are processed through an aspect-based sentiment analysis (ABSA) pipeline and analyzed alongside each cadet's network of activities, awards, and company affiliations.

Developed as part of an undergraduate thesis exploring the social structure of PMA Class of 1980 through the lens of their yearbook.

[![Build Notebook](https://img.shields.io/badge/View%20Notebook-nbviewer-orange?logo=jupyter)](https://nbviewer.org/github/jpgavieta/pma-yearbook-sentiment-kg/blob/main/notebooks/sentimentkg_build.ipynb)
[![Analysis Notebook](https://img.shields.io/badge/View%20Notebook-nbviewer-orange?logo=jupyter)](https://nbviewer.org/github/jpgavieta/pma-yearbook-sentiment-kg/blob/main/notebooks/sentimentkg_analysis.ipynb)


> **Note:** This repo does not include the real yearbook data — only fictional biographies as examples of the expected `.md` input format. The image-to-text pipeline is also not included; every `.md` file was ultimately transcribed manually due to poor OCR quality.

---

## Repository structure

```
pma-yearbook-sentiment-kg/
│
├── data/
│   ├── raw/                         # Input .md files (example profiles only)
│   │   ├── example_profile1.md
│   │   └── example_profile2.md
│   └── processed/                   # ETL outputs — gitignored (real data stays local)
│
├── envs/
│   ├── env_sentimentkg_build.yml    # Conda env for the build notebook (Python 3.7)
│   └── env_sentimentkg_analysis.yml # Conda env for the analysis notebook (Python 3.12)
│
├── notebooks/
│   ├── sentimentkg_build.ipynb      # Step 2: Aspect extraction + sentiment scoring
│   └── sentimentkg_analysis.ipynb   # Step 3: Network analysis + visualizations
│
├── src/
│   └── profiledata_etl.py           # Step 1: Markdown → JSON → CSV
│
├── figures/                         # Saved plot outputs — gitignored
├── .gitignore
└── README.md
```

---

## Pipeline overview

```
data/raw/  (.md files, one per graduate)
        ↓  src/profiledata_etl.py
  data/processed/  (merged JSON + CSV)
        ↓  notebooks/sentimentkg_build.ipynb
  Aspect extraction (spaCy) + Sentiment scoring (ABSA)
        ↓  notebooks/sentimentkg_analysis.ipynb
  Sentiment distributions + Network graphs
```

---

## Environments

This project uses two separate conda environments — one per notebook — because the ABSA
library requires Python 3.7, while the analysis stack runs on modern Python.

```bash
# Build notebook
conda env create -f envs/env_sentimentkg_build.yml
conda activate sentimentkg-build
python -m spacy download en_core_web_sm

# Analysis notebook
conda env create -f envs/env_sentimentkg_analysis.yml
conda activate sentimentkg-analysis
```

See the `.yml` files in `envs/` for the full dependency list of each environment.

---

## Usage

### Step 1 — Run the ETL pipeline

Parses all `.md` files in a folder and produces one merged JSON and one merged CSV.

```bash
python src/profiledata_etl.py --input data/raw/ --output data/processed/ --name sword80
```

Output:
```
data/processed/
  ├── sword80.json
  └── sword80.csv
```

No external dependencies — uses Python stdlib only (`re`, `ast`, `csv`, `json`, `argparse`).

### Step 2 — Aspect extraction + sentiment scoring

Activate the `sentimentkg-build` environment, then open and run `notebooks/sentimentkg_build.ipynb`.

Requires the [Aspect-Based Sentiment Analysis](https://github.com/ScalaConsultants/Aspect-Based-Sentiment-Analysis) library installed as an editable package:

```bash
git clone https://github.com/ScalaConsultants/Aspect-Based-Sentiment-Analysis.git
cd Aspect-Based-Sentiment-Analysis
pip install -e .
```

This notebook:
- Splits graduate text into `Description` (perceived sentiment) and `Quote` (self-expressed sentiment)
- Extracts aspect candidates using spaCy dependency parsing
- Scores each aspect's sentiment using a BERT-based ABSA classifier
- Builds activity and company affiliation matrices for network analysis

### Step 3 — Network analysis + visualizations

Activate the `sentimentkg-analysis` environment, then open and run `notebooks/sentimentkg_analysis.ipynb`.

This notebook answers three research questions:
1. **What sentiments exist?** — polarity scores and sentiment type distributions across perceived and self-expressed text
2. **What networks exist?** — one-mode co-activity graph and two-mode cadet–activity bipartite graph
3. **How do sentiments affect networks?** — sentiment coherence and centrality profiles of the most connected cadets

---

## Input format

The pipeline expects one `.md` file per graduate, structured like this:

```markdown
## Name
- JOSE DELA CRUZ SANTOS
## Nickname
- Joey
- Santito
## Hometown
- San Miguel, Bulacan
## Birthdate
- 3 March 1957
## Education
- San Miguel National High School, San Miguel, Bulacan
## Company
- Delta
## Activities
- **Member** - Track and Field Club
- **Cadet-in-Charge** - Catholic Action Group
## Awards
- Dean's List
- **Major "A"** - Karate
## Qualifications
- **Marksman** - US Rifle Caliber 7.62 mm M14
## Description
- Quiet in word but loud in deed...
## Quote
- A man who stands for nothing will fall for anything.
```

See `data/raw/example_profile1.md` and `data/raw/example_profile2.md` for full examples.

---

## Data fields

Each graduate record contains:

| Variable | Description |
|---|---|
| `id` | Unique cadet identifier (e.g. `016-80`) |
| `Name` | Full name |
| `Nickname` | Comma-separated list of nicknames |
| `Hometown` | City/province of origin |
| `Birthdate` | Date of birth |
| `Education` | Pre-academy school |
| `Company` | Cadet company (Alfa through Hawk) |
| `Activities.*` | Role held in each activity or organization |
| `Awards.*` | Athletic letter grades and academic honors |
| `Qualifications.*` | Marksmanship ratings per weapon |
| `Description` | Third-person profile written by peers |
| `Quote` | Self-chosen quote |

`Activities`, `Awards`, and `Qualifications` are pivoted wide — each unique activity, award, or weapon becomes its own column, with the role or rating as the cell value and `N/A` for graduates who were not involved.