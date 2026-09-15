# 🌌 Astrobit — AI Exoplanet Detection

> **Team Stark | Exoplanet Transit Detection from Stellar Light Curves**

Astrobit is an exoplanet detection pipeline designed to identify possible **transiting exoplanets** from stellar light-curve data.

The project analyzes variations in a star's brightness and searches for periodic transit-like signals. A transit occurs when a planet passes in front of its host star and causes a small, repeated decrease in observed brightness.

The pipeline combines **light-curve preprocessing, Box Least Squares (BLS) period searching, candidate characterization, confidence scoring, and submission validation** to produce a structured prediction for each target star.

---

## 🚀 Project Goal

The primary objective of Astrobit is to answer:

> **Does this star show evidence of a transiting planet?**

For every target star, the pipeline produces:

* `prediction` — whether a transit candidate was detected
* `confidence` — confidence score between 0 and 1
* `period` — estimated orbital period in days
* `depth_ppm` — estimated transit depth in parts per million
* `duration_hours` — estimated transit duration in hours

The final predictions are exported as a competition-ready CSV submission.

---

## 🔭 How Transit Detection Works

When a planet passes between its host star and the observer, the measured brightness of the star temporarily decreases.

If this decrease:

1. has a transit-like shape,
2. repeats periodically,
3. and is sufficiently significant compared with the surrounding noise,

it can be evidence of a transiting exoplanet.

Astrobit searches the light curve for these repeating box-shaped dips using **Box Least Squares (BLS)**.

---

## 🧠 Pipeline Overview

### [Astrobit Exoplanet Detection Pipeline]
<img width="994" height="651" alt="image" src="https://github.com/user-attachments/assets/743b3185-8c9a-4ab6-984a-1ba9e7dcc735" />


### Pipeline Steps

```text
Light Curve
     ↓
Preprocessing
     ↓
Coarse-to-Fine BLS Search
     ↓
Transit Candidate
     ↓
Period / Depth / Duration
     ↓
Confidence Score
     ↓
Prediction
     ↓
Submission CSV
     ↓
Validation
```

### 1. Input Light Curves

The pipeline receives time-series observations of stellar brightness.

Each light curve contains measurements representing how the observed flux of a star changes over time.

The project uses training/development data for pipeline development and evaluates the final pipeline on the private target set.

---

### 2. Preprocessing

Before searching for a transit signal, the light-curve data is prepared for analysis.

Typical processing includes:

* extracting time and flux measurements
* handling the light-curve arrays
* normalizing the flux
* preparing the signal for periodic transit detection

Good preprocessing is important because transit signals can be extremely shallow compared with stellar and observational noise.

---

### 3. Box Least Squares Search

The core detection method is **Box Least Squares (BLS)**.

BLS is particularly suitable for transit detection because a planetary transit can approximately resemble a box-shaped decrease in brightness.

Astrobit uses a **coarse-to-fine BLS search**:

```text
Broad period search
        ↓
Identify promising region
        ↓
Fine period search
        ↓
Select best candidate
```

This allows the pipeline to efficiently search for periodic signals while refining the most promising candidate.

---

### 4. Candidate Characterization

For a detected candidate, Astrobit estimates the properties of the transit signal.

#### Orbital Period

The estimated time between repeating transit events.

**Unit:** days

#### Transit Depth

The approximate reduction in stellar brightness during transit.

**Unit:** parts per million (ppm)

#### Transit Duration

The approximate amount of time the star remains in transit.

**Unit:** hours

These measurements are included in the final submission for detected candidates.

---

### 5. Confidence Scoring

Astrobit assigns every star a continuous confidence value between:

```text
0 ≤ confidence ≤ 1
```

The confidence score is important because the competition evaluates not only the hard `prediction`, but also how well the stars are **ranked by likelihood of containing a transit**.

Therefore, the pipeline avoids assigning the same confidence value to every target.

A high-confidence candidate should correspond to stronger transit evidence, while uncertain candidates receive lower confidence scores.

---

### 6. Detection Decision

The pipeline converts the candidate analysis into a binary prediction:

```text
prediction = 1
```

when the pipeline considers the star a credible transit candidate.

Otherwise:

```text
prediction = 0
```

For non-detections, the characterization fields are intentionally left empty.

---

## 📊 Final Submission Format

The competition requires exactly **87 target stars**.

The final CSV contains:

| Column           | Description                             |
| ---------------- | --------------------------------------- |
| `star_id`        | Unique target identifier                |
| `prediction`     | `1` for detected transit, `0` otherwise |
| `confidence`     | Confidence score from 0 to 1            |
| `period`         | Detected orbital period in days         |
| `depth_ppm`      | Transit depth in ppm                    |
| `duration_hours` | Transit duration in hours               |

Example:

```csv
star_id,prediction,confidence,period,depth_ppm,duration_hours
STAR_0000,1,0.94,35.4184,1976.0,6.18
STAR_0001,0,0.03,,,
STAR_0002,1,0.61,12.3341,285.5,3.92
```

For `prediction = 0`, the following fields are left empty:

```text
period
depth_ppm
duration_hours
```

---

## ✅ Submission Validation

Before submission, the generated CSV is validated for:

* exactly 87 data rows
* exactly 6 required columns
* unique `star_id` values
* valid `STAR_####` identifiers
* predictions restricted to `0` or `1`
* confidence values between `0` and `1`
* valid positive periods for detected candidates
* complete characterization values for positive predictions
* empty characterization fields for non-detections

The final Team Stark submission contains:

```text
43 detections
44 non-detections
87 unique confidence values
```

The confidence column therefore provides a continuous ranking rather than a flat set of scores.

---

## 📈 Evaluation Strategy

The competition evaluates Astrobit in two major areas.

### Detection Performance

The binary predictions are evaluated using:

* Precision
* Recall
* F1 Score

The confidence ranking is evaluated using:

* PR-AUC
* Average Precision

This means that identifying the correct stars is important, but **ranking stronger candidates above weaker candidates is also important**.

---

### Characterization Performance

For stars where a known transit exists and Astrobit predicts `1`, the detected signal is further evaluated.

The main characterization quantities are:

* orbital period accuracy
* transit depth accuracy

Period matching allows certain aliases such as:

* true period
* 2× period
* ½ period
* 3× period

within the specified tolerance.

Because of this, Astrobit reports measured candidate parameters rather than arbitrary values.

---

## 🏗️ Project Architecture

```text
                    ┌─────────────────────┐
                    │   Stellar Data      │
                    │    Light Curves     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Preprocessing     │
                    │ Normalization/Clean │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     BLS Search      │
                    │ Coarse → Fine       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Candidate Signal    │
                    │    Detection        │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
       ┌──────────────────┐        ┌──────────────────┐
       │ Characterization │        │ Confidence Score │
       │ Period / Depth / │        │     0 → 1        │
       │    Duration      │        └────────┬─────────┘
       └────────┬─────────┘                 │
                └─────────────┬────────────┘
                              ▼
                    ┌─────────────────────┐
                    │ Final Prediction    │
                    │      0 or 1         │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Submission CSV      │
                    │ submission_teamstark│
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Validation & Submit │
                    └─────────────────────┘
```

---

## 🧪 Development Workflow

The project was developed iteratively:

```text
Dataset
   ↓
Explore Light Curves
   ↓
Build Initial BLS Search
   ↓
Improve Search Strategy
   ↓
Coarse-to-Fine Search
   ↓
Generate Candidate Parameters
   ↓
Assign Confidence
   ↓
Generate Submission
   ↓
Validate Submission
```

The search component was improved from a basic BLS search toward a more refined **coarse-to-fine candidate search**, with the goal of improving both detection and characterization.

---

## 📁 Repository Structure

```text
Exoplanet-detection
│
├── .gitignore
├── README.md
├── requirements.txt
│
├── Astrobit_Transit_Detection.ipynb
│
├── train_labels.csv
├── train_truth.csv
├── dev_labels.csv
├── dev_truth.csv
│
├── submission_astrobit.csv

```

> The exact filenames may vary depending on the final repository organization.

---

## 🛠️ Technologies Used

* **Python**
* **NumPy** — numerical computation
* **Pandas** — data handling and CSV generation
* **Astropy** — astronomical/time-series analysis
* **Box Least Squares (BLS)** — periodic transit search
* **Jupyter / Google Colab** — development and experimentation
* **Git & GitHub** — version control and project submission

---

## 💡 Why BLS?

Transit signals are generally characterized by a relatively short decrease in brightness followed by a return to the baseline level.

BLS searches for periodic box-shaped patterns and therefore provides a strong baseline for detecting transit-like signals in stellar time-series data.

Astrobit further improves this process through a coarse-to-fine search strategy rather than relying only on a single broad search.

---

## 🎯 Current Result

The current submission pipeline successfully generates a valid competition submission containing:

```text
87 target stars
43 predicted detections
44 predicted non-detections
87 unique confidence values
```

The generated submission file is:

```text
submission_teamstark.csv
```

---

## 👥 Team

### Team Stark

**Project:** Astrobit

**Task:** AI-assisted exoplanet transit detection

**Output:** Competition-ready exoplanet detection submission

---

## 🔮 Future Improvements

Potential improvements include:

* stronger detrending and noise removal
* improved stellar variability handling
* transit candidate vetting
* automated alias detection
* better confidence calibration
* ensemble-based candidate scoring
* improved shallow-transit detection
* more robust period and depth estimation
* additional astronomical validation checks

The main long-term goal is to improve sensitivity to **shallow transit signals near the noise floor** while maintaining high precision on strong candidates.

---

## 📜 License

This project is intended for educational, research, and competition purposes.

If this repository is reused or extended, please provide appropriate attribution to the original project and contributors.
