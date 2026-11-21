# 🎼 Spotify Music Recommendation System (End-to-End)

This project implements a **content-based music recommendation engine**.  
It transforms a **personal Spotify playlist** into a **unique mathematical profile**, then compares this profile to a historical catalog of **160,000 songs** to identify the tracks that best match the user's **current musical tastes**.

The goal is to connect **real listening data** with **interpretable machine learning methods**, without relying on the behavior of other users.

---

## 🛠 1. Configuration and Dependencies

The project relies on several specialized libraries:

### Data Processing and Manipulation
- `pandas`
- `numpy`
- `itertools`

### Machine Learning
- `scikit-learn`
  - `TfidfVectorizer`
  - `MinMaxScaler`
  - `cosine_similarity`

### Visualization
- `matplotlib`
- `skimage`

### Spotify API
- `spotipy`  
  Used for OAuth authentication and extracting user data (playlists, metadata).

---

## 📊 2. Dataset Exploration and Cleaning (Kaggle)

### Analysis of `data.csv` — Song Level
This file contains the **audio features** of each track:
- acousticness
- danceability
- energy
- tempo
- etc.

#### Duplicate Problem
The same song can appear multiple times with different IDs.

**Solution Implemented:**
- Creation of a unique key `artists_song` (Artist + Song Name)
- Removal of duplicates
- Retention of the most popular or most recent version

Objective: prevent multiple versions of the same title from biasing recommendations.

---

### Analysis of `data_w_genres.csv` — Artist Level
**Musical genres**, essential for content-based recommendations, are not present in the main file.

#### Regex Cleaning
Genres are stored as character strings:

The code uses:
- `re.findall`
- `re.sub`

to transform these strings into **exploitable Python lists**:

#### Enrichment by Explosion
- Use of `.explode()` on the artist column
- Enables a **clean merge** between:
  - songs
  - genres associated with each artist

---

## 🧬 3. Feature Engineering — Mathematical Transformation

The `create_feature_set` function is the **core of the model**.  
It transforms each song into a **comparable numerical vector**.

---

### TF-IDF Applied to Genres
Instead of simple counting, **TF-IDF** is used.

**Why?**
- Very common genres (ex: `pop`) are not discriminating
- Rare genres (ex: `jazz fusion`, `neo soul`) better define musical identity

TF-IDF:
- penalizes frequent terms
- valorizes distinctive terms
- improves recommendation precision

---

### Normalization and Weighting

#### Min-Max Scaling
Audio variables (ex: tempo ≈ 120) are scaled to **[0, 1]** in order to:
- make all variables comparable
- prevent a single dimension from dominating similarity calculation

#### Category Weighting
Not all features have the same importance:

- **Genres (TF-IDF)** → dominant weight
- **Year (`year`)** → multiplied by `0.5`
- **Popularity** → multiplied by `0.15`

These coefficients reflect the relative importance of each dimension in musical perception.

---

## 🔌 4. Spotify API Integration

The Spotify API is used via **Spotipy** to integrate user data.

### Extraction
The `create_necessary_outputs` function:
- iterates through the selected playlist
- retrieves Spotify IDs of songs
- extracts essential metadata

### Validation
Only songs:
- present in the Spotify playlist
- **and** existing in the Kaggle dataset (160k)

are kept to build the user profile.

### Recency
The added date (`added_at`) is saved to introduce a **temporal bias** in the recommendation.

---

## 🧠 5. Recommendation Logic

### Recency Weighting (Recency Bias)
The `generate_playlist_feature` function:

- calculates the number of months elapsed since the most recently added song
- applies **exponential weighting**

Result:
- recently added songs primarily define the profile
- older tastes have decreasing influence

---

### Profiling — The Profile Vector
All weighted songs in the playlist are:
- added together
- averaged

This produces a **unique vector**, representing the user's **current musical profile**  
(their "musical portrait").

---

### Matching — Cosine Similarity
The `generate_playlist_recos` function:

- compares the profile vector to all titles in the dataset
- excludes songs already present in the playlist

**Cosine similarity** measures the angle between two vectors:
- score close to `1` → strong similarity
- score close to `0` → weak match

---

## 🖼 6. Visualization

The `visualize_songs` function:
- downloads album covers via Spotify URLs
- displays the **40 best recommendations**
- uses a grid of subplots with corresponding titles

This step enables **immediate visual reading** of results.

---

## ⚠️ About Data Files

**Note:** The dataset CSV files (`.csv` and `.json`) are **not included** in this repository due to their large size (> 100 MB), which exceeds GitHub's file size limits.

### Where to Find the Data

Download the datasets from Kaggle:
1. **Spotify Dataset (160k tracks)**  
   https://www.kaggle.com/datasets/yamaerenay/spotify-dataset-1921-2020-160k-tracks
   
2. **Spotify Dataset (600k tracks)**  
   https://www.kaggle.com/datasets/yamaerenay/spotify-dataset-19212020-600k-tracks

Extract the files and place them in these folders:
```
project_root/
├── data/              (from 600k dataset)
│   ├── artists.csv
│   ├── dict_artists.json
│   └── tracks.csv
└── data_160k/         (from 160k dataset)
    ├── data.csv
    ├── data_by_artist.csv
    ├── data_by_genres.csv
    ├── data_w_genres.csv
    └── data_by_year.csv
```

---

## 🚦 How to Run the Project

### 1. Spotify API Configuration

> **Note:** This project was first developed in a private repository. The Spotify credentials visible in earlier commits have been revoked and no longer work; use your own credentials as described below.

- Create an application on **Spotify for Developers**
- Retrieve your `CLIENT_ID` and `CLIENT_SECRET`, then export them as environment variables before launching Jupyter:

```bash
export SPOTIPY_CLIENT_ID=your_client_id
export SPOTIPY_CLIENT_SECRET=your_client_secret
```

### 2. Data
Download the required CSV files from Kaggle (see **About Data Files** section above) and extract them to:
- `data/` folder
- `data_160k/` folder

### 3. Execution
1. Run the data preparation cells (cleaning, TF-IDF, OHE)
2. Connect to Spotify via Spotipy
3. Execute the recommendation function
4. Display the final gallery of album covers

---

## 📚 Context
Project completed as part of **data analysis** and the development of a  
**content-based music recommendation system**.
