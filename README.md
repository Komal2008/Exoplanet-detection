# Exoplanet-detection
AI-powered detection and characterization of Earth-like exoplanets from Kepler photometry using signal processing and machine learning.

## Problem

Earth-like exoplanets produce extremely shallow transit signals that can be hidden by stellar variability, instrumental systematics, and observational noise.

This project aims to recover these weak signals using a combination of:

- Noise and stellar variability modelling
- Transit-preserving detrending
- Coarse-to-fine period search
- Box Least Squares (BLS)
- Phase folding and transit stacking
- Machine learning-based candidate vetting
- Confidence calibration

## Pipeline

Raw Kepler SAP Flux  
→ Noise Modelling  
→ Detrending  
→ Transit Detection  
→ Candidate Generation  
→ ML Classification  
→ Candidate Ranking

## Tech Stack

- Python
- Google Colab
- NumPy
- Pandas
- SciPy
- Astropy
- Scikit-learn
- XGBoost
- Matplotlib
- GitHub

## Dataset

The project uses Kepler photometry provided by the hackathon organizers.

The dataset is not stored directly in this repository.

## Project Status

🚧 Development in progress.
