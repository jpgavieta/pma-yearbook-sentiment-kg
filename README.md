# Rule-Based ABSA Knowledge Graph for PMA Graduate Profiles

A custom rule-based NLP data pipeline based on Philippine Military Academy (PMA) Class of 1980 yearbook records. The yearbook pages were transcribed to structured text (one MD per student), then processed through an aspect-based sentiment analysis (ABSA) pipeline to study the relationship between a cadet's social network and the sentiment expressed in their written profiles.

This repo was developed as part of my undergrad thesis exploring the social structure of PMA Class of 1980 through the lens of their yearbook. 

In a yearbook, each student's profiles — which includes peer-authored descriptions and self-chosen quotes — are treated as dual windows into the external perceptions and internal self-expressions of identity; analyzed together with each cadet's network of activities, awards, and company affiliations.

---

## What's in this repo

```
pma-yearbook-sentiment-kg/
│
├── data/
│   ├── example_profile1.md       # Fake graduate sample
│   ├── example_profile2.md       # Fake graduate sample 
│   └── graduates.csv              # Final merged dataset (real data not included)
│
├── profiledata_etl.py             # Step 1–5: Markdown → JSON → CSV
├── sentimentkg_build.ipynb        # Step 6: Aspect extraction + sentiment scoring
├── sentimentkg_analysis.ipynb     # Step 7: Network analysis + visualizations
│
└── README.md
```

> **NOTE:** This repo does not include the real yearbook data, only FICTIONAL biographies as examples for the input `.md` format. 
> Also, this repo does not include the image-to-text processing pipeline because I'm not proud of it. In the end, I had to manually rewrite every `.md` for for every student profile because the OCR extraction was so poor. 
> This was my very first python project!


---

## Pipeline overview

```
Markdown (md) for each student profile 
        ↓  profiledata_etl.py
  Merged JSON + CSV
        ↓  sentimentkg_build.ipynb
  Aspect extraction (spaCy) + Sentiment scoring (ABSA)
        ↓  sentimentkg_analysis.ipynb
  Network graphs + Sentiment distribution plots
```

---

## Data Fields (Variables)

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

The `Activities`, `Awards`, and `Qualifications` columns are pivoted wide — each unique activity/award/weapon becomes its own column, with the role or rating as the cell value and `N/A` for graduates who weren't involved.

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

See `data/example_graduate_1.md` and `data/example_graduate_2.md` for full examples.

---

## Usage

### Step 1: Run the data pipeline

```bash
python graduate_pipeline.py --input <md_folder> --output <output_folder> --name <dataset_name>
```

**Example:**
```bash
python graduate_pipeline.py --input data/ --output results/ --name graduates
```

**Output:**
```
results/
  ├── graduates.json
  └── graduates.csv
```

No external dependencies — uses Python stdlib only (`re`, `ast`, `csv`, `json`, `argparse`).

### Step 2: Aspect extraction + sentiment scoring

Open and run `sentinet_code_preprocess.ipynb`.

Requires the [Aspect-Based Sentiment Analysis](https://github.com/ScalaConsultants/Aspect-Based-Sentiment-Analysis) library. Setup:

```bash
git clone https://github.com/ScalaConsultants/Aspect-Based-Sentiment-Analysis.git
cd Aspect-Based-Sentiment-Analysis
conda env create -f environment.yml
conda activate Aspect-Based-Sentiment-Analysis
pip install -e .
python -m spacy download en_core_web_sm
```

This notebook:
- Splits graduate text into `Description` (perceived sentiment) and `Quote` (self-expressed sentiment)
- Extracts noun-based aspect candidates using spaCy dependency parsing
- Scores each aspect's sentiment using BERT-based ABSA
- Builds activity/company affiliation matrices for network analysis

### Step 3: Network analysis + visualizations

Open and run `sentinet_code_analysis.ipynb`.

This notebook answers three research questions:
1. **What sentiments exist?** — polarity scores, sentiment type distribution (positive / negative / mixed / neutral)
2. **What networks exist?** — bipartite graphs of cadets and their activities, weighted by role
3. **How do sentiments affect networks?** — affective social reputation (sentiment × network centrality)

---

## Dependencies

| Package | Used in |
|---|---|
| `pandas` | All notebooks |
| `spacy` | Preprocessing notebook |
| `transformers` | Preprocessing notebook (ABSA) |
| `networkx` | Analysis notebook |
| `matplotlib` | Analysis notebook |
| `aspect_based_sentiment_analysis` | Preprocessing notebook |

Python version: **3.7.16** (required by the ABSA library)
