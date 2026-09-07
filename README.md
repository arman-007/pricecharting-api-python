# PriceCharting Python API & Scraper — Historical Sales, PSA/BGS Graded Comps & Game Valuations

[![Apify Actor](https://img.shields.io/badge/Apify%20Actor-incognito__mode%2Fpricecharting--product--scraper-blue?logo=apify)](https://apify.com/incognito_mode/pricecharting-product-scraper)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/arman-007/pricecharting-api-python)

A production-ready Python client, CLI utility, and market analytics suite for extracting comprehensive pricing data, historical sales time-series, PSA/BGS/CGC/TAG/ACE graded comps, POP reports, and 1600px high-resolution images from [PriceCharting](https://www.pricecharting.com) via the Apify Actor: **[`incognito_mode/pricecharting-product-scraper`](https://apify.com/incognito_mode/pricecharting-product-scraper)**.

---

## 🎯 Why This Exists: The Problem with the Official API

| Feature | Official PriceCharting API | This Scraper / Wrapper |
|---|---|---|
| **Monthly Subscription** | **$49.00 / month** minimum | **$0 / month** (Apify free tier covers hundreds of items) |
| **Price History Time Series** | ❌ **Omitted** (today's price only) | ✅ **Full historical time series** per condition |
| **High-Res Images** | ❌ **Omitted** | ✅ **Full 1600px photos** (box, manual, card front/back) |
| **Complete Grading Ladder** | ❌ Basic tiers only | ✅ **Every grade**: TAG 10, ACE 10, CGC Pristine, BGS Black Label |
| **Sold Comps & POP Reports** | ❌ Omitted | ✅ **eBay/TCGPlayer sold listings** & PSA/CGC POP report |
| **Failed Lookup Billing** | ⚠️ Billed against monthly quota | ✅ **Zero cost** on failed/missing lookups |

---

## 🚀 Quickstart (Python)

### 1. Installation

```bash
git clone https://github.com/arman-007/pricecharting-api-python.git
cd pricecharting-api-python
pip install -r requirements.txt
```

### 2. Extract Data in 4 Lines of Code

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.getenv("APIFY_API_TOKEN"))

run_input = {
    "products": [
        "https://www.pricecharting.com/game/pokemon-base-set/charizard-4",
        "https://www.pricecharting.com/game/gameboy-advance/pokemon-emerald",
        "7141" # Direct numeric product ID
    ],
    "scrapeDetails": True,
    "includeRecentSales": False
}

run = client.actor("incognito_mode/pricecharting-product-scraper").call(run_input=run_input)
dataset_items = list(client.dataset(run["defaultDatasetId"]).iterate_items())

for item in dataset_items:
    print(f"{item['productName']}: Loose=${item['prices']['loose']} | PSA 10=${item['prices']['manualOnly']}")
```

---

## 📊 Google Sheets Live Integration (Zero Code)

No Python knowledge required. You can stream live PriceCharting market valuations directly into Google Sheets:

```excel
=IMPORTDATA("https://api.apify.com/v2/datasets/<DATASET_ID>/items?format=csv")
```

Whenever the actor executes its scheduled run, your spreadsheet recalculates automatically.

---

## 🏷️ Price Slot Mapping Reference

PriceCharting maps 6 unified columns across all categories. This wrapper normalizes them into stable keys:

| JSON Key | Video Games | Trading Cards (Pokémon / MTG / Sports) |
|---|---|---|
| `prices.loose` | Loose (cartridge / disc) | **Ungraded (Raw NM)** |
| `prices.cib` | Complete in Box | Grade 7 |
| `prices.new` | Factory Sealed | Grade 8 |
| `prices.graded` | Graded Sunk Box | Grade 9 |
| `prices.boxOnly` | Box Only | Grade 9.5 |
| `prices.manualOnly` | Manual Only | **PSA 10 Gem Mint** |

For company-specific grades (BGS 10 Black Label, CGC 10 Pristine, TAG 10, ACE 10, SGC 10), use the **`fullPrices`** object where labels are delivered verbatim.

---

## 📈 Jupyter Notebook Market Analytics

A pre-built analytics notebook is available under [`notebooks/pricecharting_market_analysis.ipynb`](notebooks/pricecharting_market_analysis.ipynb):

1. **Grading Multiplier Calculation**:
   ```python
   multiplier = psa10_price / raw_price
   ```
   Automates profitability analysis before submitting raw cards to PSA/CGC.
2. **Sum-of-Parts Video Game Assembly**:
   Calculates whether buying Loose + Box + Manual separately is cheaper than purchasing an assembled CIB copy.
3. **Time-Series Charting**:
   Visualizes multi-year appreciation curves for high-demand collectibles.

Run the notebook:
```bash
jupyter notebook notebooks/pricecharting_market_analysis.ipynb
```

---

## 💻 CLI Usage

The repository includes a ready-to-run CLI tool:

```bash
# 1. Run offline against bundled samples (no API token required)
python extract_prices.py --sample charizard_base_set --spread

# 2. Extract live data by URL or numeric ID
python extract_prices.py --products https://www.pricecharting.com/game/pokemon-base-set/charizard-4 6861 --output market_comps.csv

# 3. Compute PSA 10 grading arbitrage margins
python extract_prices.py --sample charizard_base_set --spread
```

---

## 📦 Output JSON Schema Sample

```json
{
  "productId": 630417,
  "productName": "Charizard #4",
  "consoleName": "Pokemon Base Set",
  "category": "pokemon-cards",
  "url": "https://www.pricecharting.com/game/pokemon-base-set/charizard-4",
  "releaseDate": "January 9, 1999",
  "imageUrl": "https://storage.googleapis.com/images.pricecharting.com/hpgpcpsd42huitud/1600.jpg",
  "images": [
    "https://storage.googleapis.com/images.pricecharting.com/hpgpcpsd42huitud/1600.jpg",
    "https://storage.googleapis.com/images.pricecharting.com/kmwn5qjyipwzbuwm/1600.jpg"
  ],
  "prices": {
    "loose": 338.42,
    "cib": 749.50,
    "new": 1199.03,
    "graded": 3175.04,
    "boxOnly": 3403.50,
    "manualOnly": 30085.73
  },
  "fullPrices": {
    "Ungraded": 338.42,
    "PSA 10": 30085.73,
    "BGS 10": 39111.00,
    "BGS 10 Black": 195555.00,
    "CGC 10 Pristine": 27475.00
  },
  "salesVolume": {
    "Ungraded": 48,
    "PSA 10": 30
  },
  "priceHistory": {
    "used": [
      { "date": "2024-01-01", "price": 295.00 },
      { "date": "2026-01-01", "price": 338.42 }
    ],
    "manualOnly": [
      { "date": "2024-01-01", "price": 24500.00 },
      { "date": "2026-01-01", "price": 30085.73 }
    ]
  }
}
```

---

## 🔗 Related Resources & Deep-Dives

- **Apify Actor**: [PriceCharting Product Scraper](https://apify.com/incognito_mode/pricecharting-product-scraper)
- **Technical Guide (Hashnode)**: [Extracting PriceCharting Market Data: Historical Sales, Graded Comps & Game Valuations into JSON](https://armanhosen.hashnode.dev/extracting-pricecharting-market-data-historical-sales-graded-comps-psa-bgs-cgc-and-game-card-valuations-into-json)
- **Substack Deep-Dive**: [Extracting PriceCharting Market Data](https://armanhosen.substack.com/p/extracting-pricecharting-market-data)
- **Twitter / X Discussion**: [@armanirfan007 Thread on Historical Comps & PSA Arbitrage](https://x.com/armanirfan007/status/2097049451978523038)

---

## 📄 License

Distributed under the MIT License. See [`LICENSE`](LICENSE) for more details.

**Author**: [Arman Hosen](https://github.com/arman-007)
