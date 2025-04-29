# MIST4610-G1-P2
This will be used for our tableau visualization group project. 
# MIST4610 Group Project 2

## Team Name and Members

### Group 1
- Caleb Sandler [@csandlerdev](https://github.com/CSandler-Dev/MIST4610-G1-P1)
- Nikita Brahmane [@nikitabrahmane](https://github.com/nikitabrahmane)
- Jimmy Vu [@jimmyvu0223](https://github.com/jimmyvu0223)
- John Housman [@jhousman1](https://github.com/jhousman1)

https://www.kaggle.com/datasets/adampq/coffee-quality-with-locations-of-origin?select=Coffee_Qlty_By_Country.csv
https://wits.worldbank.org/trade/comtrade/en/country/ALL/year/2018/tradeflow/Imports/partner/WLD/product/090111#

### Description of dataset
  The "Coffee Quality with Locations of Origin" data set provides a comprehensive overview of Arabica and Robusta coffee samples from around the world. The data set contains 1,318 rows and 28 columns, and each row represents a unique coffee sample that has been graded and evaluated and graded by expert coffee tasters. The data set was compiled by the Coffee Quality Institute and includes both coffees' sensory evaluation values and metadata about the coffee’s origin.
  The columns in the data include detailed sensory scores such as Aroma, Flavor, Aftertaste, Acidity, Body, Balance, Uniformity, Sweetness, all scored as numerical values ranging from 0 to 10 scale. The data set also includes categorical and text fields like Country of Origin, Continent of Origin, Variety, and Processing Method, which provide descriptive information about where and in what way the coffee was grown and processed. 
  Overall, the dataset is a rich source for exploring patterns in coffee quality, regional variability in coffee characteristics, and factors behind high-scoring coffee, making it valuable for data analysts and coffee enthusiasts alike.

## 1  Datasets Used

| # | Dataset | Rows × Cols | Core Fields Kept | Why We Picked It |
|---|---------|-------------|------------------|------------------|
| 1 | **Coffee_Qlty.csv** (Kaggle) | 1 339 × 28 | Country, 9 sensory scores, defect counts, Processing Method | Only open cupping dataset with both flavour and origin metadata. |
| 2 | **price_2018.csv** (UN Comtrade / WITS) | 34 × 4 after clean | Reporter Country, Trade Value (US$ 1000), Net-weight (kg) | Gives **actual 2018 market price** per origin—essential for a value metric. |
| 3 | **altitude_country.csv** (NASA SRTM + CIA) | 190 × 2 | Country, Mean Elevation (m) | Altitude is a well-known driver of Arabica quality; lets us test that belief. |

---

## 2  What We’re Trying to Show

Specialty roasters care about **quality per dollar**, not raw cupping scores.  
Our goal is to identify:

1. **Origins** where high cup quality is **cheap** relative to peers.  
2. **Processing methods** that further improve key flavour attributes **without adding cost**.

Altitude is analysed as a possible “built-in advantage” explaining why some origins naturally outperform.

---

## 3  Cleaning, Manipulation & Joining

| Step | Action | Rationale |
|------|--------|-----------|
| **Country harmonisation** | Created `country_std`; manual fixes for “Côte d’Ivoire”, “United States (Hawaii)”, etc. | Ensures 1-to-1 joins across tables. |
| **Price conversion** | `USD_per_kg = (TradeValue × 1000) / NetWeight` | WITS value is in thousands of USD. |
| **Kg vs Tonne bug fix** | If NetWeight \< 1 000 → already kg; else divide by 1 000. | Brazil & a few others reported tonnes. |
| **Outlier filter** | Dropped rows where price \< \$1.50/kg or \> \$40/kg | Removes remaining FOB anomalies. |
| **Processing bucket** | Regex → **Washed, Natural, Honey, Unknown** | Collapses 30+ raw strings. |
| **WSS (Weighted Sensory Score)** | Σ(Aroma…Sweetness + Uniformity + Clean Cup) – 0.5×Defects | CQI composite quality metric. |
| **CQE (Cost–Quality Efficiency)** | `WSS / USD_per_kg` | “Bang-for-buck” value indicator. |
| **AltitudeBand** | Low \< 1200 m · Mid 1200–1600 m · High \> 1600 m | Simplifies terroir comparison. |
| **Z-Score calc** | Table calc `WINDOW_ZSCORE(SUM(Measure Values))` | Shows attribute lifts in σ units. |

After cleaning we keep **851 rows across 14+ origins** with 0 % nulls in price, altitude, or process fields.  
All steps are reproducible via `scripts/build_coffee_master.py`.

---

## 4  Chain of Exploration

1. **CQE Scatter (Median WSS vs. Median Price)**  
   *Bubble size = sample count, colour = AltitudeBand.*  
   → Ethiopia, Guatemala, Colombia emerge as **high-altitude, high-CQE “sweet-spots.”**

2. **High-CQE Set**  
   Lasso those bubbles → `High_CQE_Set` for drill-down.

3. **Process vs Price Mini-Scatter** (faceted by Process)  
   → Natural lots in Ethiopia & Kenya sit **+0.8 WSS at same price**; Honey in Costa Rica shows higher price, flat WSS.

4. **Flavour Z-Score Bars** (filtered by country & process)  
   → Natural Ethiopians **+1.6 σ Aroma, +1.4 σ Sweetness**; Honey adds Body in Costa Rica but only +0.3 σ.

These steps shaped our final questions:

---

## 5  The Two Questions & Their Importance

| # | Question | Tableau View | Why It Matters |
|---|----------|--------------|----------------|
| **Q1** | **Which producing countries offer the best Cost-Quality Efficiency (CQE), and how does altitude explain those rankings?** | CQE scatter with AltitudeBand colour & interactive set | Guides sourcing: buyers can prioritise high-CQE, high-altitude origins to maximise quality per dollar. |
| **Q2** | **Within high-CQE origins, which processing method most improves key flavour attributes without increasing cost?** | Z-score bar chart (filtered by High_CQE_Set & process) | Helps purchasers decide between Natural, Washed, or Honey lots; justifies paying a process premium only when flavour lift offsets cost. |

---

INTERACTIVE VISUIALIZATION ON TABLEAU PUBLIC : https://public.tableau.com/shared/XRMHFSDX5?:display_count=n&:origin=viz_share_link
