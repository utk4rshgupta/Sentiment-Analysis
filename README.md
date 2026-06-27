# Quick Commerce Sentiment Analysis

A pipeline that scrapes Google Play Store reviews for India's top quick-commerce and grocery delivery apps, runs VADER sentiment analysis on them, and visualizes the results.

## Apps Covered

| App | Package ID |
|---|---|
| Blinkit | `com.grofers.customerapp` |
| Zepto | `com.zeptoconsumerapp` |
| Big Basket | `com.bigbasket.mobileapp` |
| Swiggy Instamart | `in.swiggy.android.instamart` |
| JioMart | `com.jpl.jiomart` |

## Pipeline Overview

```
Google Play Store
      ↓
  Scraper (google-play-scraper)
      ↓
  indian_quick_commerce_reviews.csv
      ↓
  Sub-service tagging (keyword-based)
      ↓
  VADER Sentiment Analysis
      ↓
  quick_commerce_sentiment_analyzed.csv + sentiment_comparison.png
```

## Project Structure

```
├── Sentiment_Analysis.ipynb              # Main notebook
├── indian_quick_commerce_reviews.csv     # Raw scraped reviews (generated)
├── quick_commerce_sentiment_analyzed.csv # Processed dataset with sentiment labels (generated)
└── sentiment_comparison.png             # Bar chart visualization (generated)
```

## Setup

### Prerequisites

- Python 3.8+
- Jupyter Notebook or Google Colab

### Install Dependencies

```bash
pip install google-play-scraper pandas nltk matplotlib seaborn
```

NLTK's VADER lexicon is downloaded automatically on first run.

## Usage

Run the notebook cells in order:

**Cell 1 — Basic Scraper**
Fetches up to 100,000 English reviews per app from the Indian Play Store (sorted by newest) and saves them to `indian_quick_commerce_reviews.csv`.

**Cell 2 — Date-Windowed Scraper** *(recommended)*
A more controlled alternative that scrapes reviews only within a configurable date window (default: last 6 months). Adjust the `MONTHS_BACK` variable at the top of the cell.

```python
MONTHS_BACK = 6  # Change to 3, 12, etc.
```

**Cell 3 — Sentiment Analysis & Visualization**
Loads the CSV, tags reviews with sub-service labels using keyword matching, runs VADER compound scoring, classifies each review as Positive / Neutral / Negative, and outputs the final CSV and chart.

## How It Works

### Scraping
Uses [`google-play-scraper`](https://github.com/JoMingyu/google-play-scraper) to pull reviews in batches, targeting `lang='en'` and `country='in'`. The date-windowed scraper stops fetching once it reaches reviews older than the configured start date (reviews are returned newest-first).

### Sub-service Tagging
Since some apps bundle multiple services (e.g., Swiggy has food delivery and Instamart), each review's text is scanned for service-specific keywords to assign a `TargetService` label before analysis.

### Sentiment Scoring
[VADER (Valence Aware Dictionary and sEntiment Reasoner)](https://github.com/cjhutto/vaderSentiment) computes a `compound` score per review ranging from -1.0 to +1.0.

| Compound Score | Label |
|---|---|
| ≥ 0.05 | Positive |
| ≤ -0.05 | Negative |
| Between | Neutral |

## Output

- **Console** — sentiment breakdown (%) by service and average Play Store star rating per service
- **`quick_commerce_sentiment_analyzed.csv`** — full dataset with `compound_score` and `Sentiment` columns appended
- **`sentiment_comparison.png`** — grouped bar chart of sentiment distribution across all services

## Notes

- The scraper includes a polite delay (`time.sleep`) between batches to avoid hitting Google's rate limits.
- Reviews with missing content are dropped before analysis.
- VADER is optimized for short, informal social text, making it well-suited for app reviews.
- The basic scraper (Cell 1) is included for quick testing; the date-windowed scraper (Cell 2) is better for reproducible, time-bounded analysis.
