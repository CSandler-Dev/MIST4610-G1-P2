# MIST4610 Group Project 2

## 1 | Group 1
- Caleb Sandler [@csandlerdev](https://github.com/CSandler-Dev/MIST4610-G1-P1)
- Nikita Brahmane [@nikitabrahmane](https://github.com/nikitabrahmane)
- Jimmy Vu [@jimmyvu0223](https://github.com/jimmyvu0223)
- John Housman [@jhousman1](https://github.com/jhousman1)

## 2 | Data Discovery & Initial Work

### Description of datasets & join choices
  The "Coffee Quality with Locations of Origin" data set provides a comprehensive overview of Arabica and Robusta coffee samples from around the world. The data set contains 1,318 rows and 28 columns, and each row represents a unique coffee sample that has been graded and evaluated by expert coffee tasters. The data set was compiled by the Coffee Quality Institute and includes both coffees' sensory evaluation values and metadata about the coffee’s origin.
  
  This dataset is incredibly rich with datapoints however, we ran into issues deriving interesting insights into the data. As such, we decided to also incorporate export information (by country) from the UN Comtrade on coffee export volume and cost (adjusted to USD) as a comparison point. We also decided to include an altitude dataset to retrieve the average elevations of the countries we're exploring as a further explanatory variable. The goal is to connect the taste data to real-world implications.

### Datasets Used

| # | Dataset | Rows × Cols | Core Fields Kept | Why We Picked It | Link |
|---|---------|-------------|------------------|------------------|----------|
| 1 | **Coffee_Qlty.csv** (Kaggle) | 1 339 × 28 | Country, 9 sensory scores, defect counts, Processing Method | Only open cupping dataset with both flavour and origin metadata. | [Kaggle 🔗](https://www.kaggle.com/datasets/adampq/coffee-quality-with-locations-of-origin?select=Coffee_Qlty_By_Country.csv)
| 2 | **price_2018.csv** (UN Comtrade / WITS) | 34 × 4 after clean | Reporter Country, Trade Value (US$ 1000), Net-weight (kg) | Gives **actual 2018 market price** per origin—essential for a value metric. | [WITS 🔗](https://wits.worldbank.org/trade/comtrade/en/country/ALL/year/2018/tradeflow/Imports/partner/WLD/product/090111#)
| 3 | **altitude_country.csv** (NASA SRTM + CIA) | 190 × 2 | Country, Mean Elevation (m) | Altitude is a well-known driver of Arabica quality; lets us test that belief. | [NASA 🔗](https://www.earthdata.nasa.gov/data/catalog)

The finalized dataset we used for the visualization is called **!joined_final.csv** and was cleaned, joined, and calculated from the three datasets above. We used the country to join the datasets.

---

## 3 | What We’re Trying to Show

Our data explores specialty coffee and its customers' perceived opinions, so specialty roasters care about **quality per dollar**, not raw cupping scores. Given the incredibly expensive world of coffee cultivation, cross-continent shipping, and sourcing, the best places to buy coffee from are the countries that balance cost with quality.
Our goal is to identify:

1. **Origins** where high cup quality is **cheap** relative to peers. 
2. **Conditions** where coffee grows best, and possibly why coffee grown in certain countries is better.
3. **Processing methods** that lead to the best key flavour attributes from the countries with the best coffee quality compared to price.

---

## 4 | Cleaning, Manipulation & Joining

| Step | Action | Rationale |
|------|--------|-----------|
| 1. **Country harmonisation** | Created `country_std`; manual fixes for “Côte d’Ivoire”, “United States (Hawaii)”, etc. | Ensures 1-to-1 joins across tables. |
| 2. **Price conversion** | `USD_per_kg = (TradeValue × 1000) / NetWeight` | WITS value is in thousands of USD. |
| 3. **Kg vs Tonne bug fix** | If NetWeight \< 1 000 → already kg; else divide by 1 000. | Brazil & a few others reported tonnes. |
| 4. **Outlier filter** | Dropped rows where price \< \$1.50/kg or \> \$40/kg | Removes remaining FOB anomalies. |
| 5. **Processing bucket** | Regex → **Washed, Natural, Honey, Unknown** | Collapses 30+ raw strings. |
| 6. **WSS (Weighted Sensory Score)** | Σ(Aroma…Sweetness + Uniformity + Clean Cup) – 0.5×Defects | CQI composite quality metric. |
| 7. **CQE (Cost–Quality Efficiency)** | `WSS / USD_per_kg` | “Bang-for-buck” value indicator. |
| 8. **AltitudeBand** | Low \< 1200 m · Mid 1200–1600 m · High \> 1600 m | Simplifies terroir comparison. |

After cleaning we keep **851 rows across 14+ origins** with 0 % nulls in price, altitude, or process fields.  

---

## 5 | Chain of Data Exploration

Given the scattered and uncoordinated nature of the datasets we pulled together, we explored the data in layers, which was formative to our understanding of the data.

1. **CQE Scatter (Median WSS vs. Median Price)**  
   *Bubble size = sample count, colour = AltitudeBand.*  
   → Ethiopia, Guatemala, and Colombia emerge as **high-altitude, high-CQE “sweet-spots.”**

2. **High-CQE Set**  
   Lasso those bubbles → `High_CQE_Set` for drill-down.

3. **Process vs Price Mini-Scatter** (faceted by Process)  
   → Natural lots in Ethiopia & Kenya sit **+0.8 WSS at the same price**; Honey in Costa Rica shows a higher price, flat WSS.

4. **Flavour Z-Score Bars** (filtered by country & process)  
   → Natural Ethiopians **+1.6 σ Aroma, +1.4 σ Sweetness**; Honey adds Body in Costa Rica but only +0.3 σ.

These steps shaped our final questions and the insights we were able to derive from the dataset:

---

## 6 | The Two Questions & Their Importance

### **Question 1 — Cost-Quality Efficiency by Origin**  
**What we ask**  
> *Which producing countries deliver the best **Cost-Quality Efficiency** (CQE = median WSS ÷ median USD per kg), and what role does altitude play in those rankings?*

#### **How we visualise it**  
* **Scatterplot** of median WSS (y) vs. median price USD/kg (x, log-scale to normalize the scale).
* **Interactive set** – lassoing the high-value quadrant creates **High_CQE_Set** for drill-downs.

#### **Why it matters**  
* Gives roasters a short-list of origins that maximise quality per dollar.  
* Altitude overlay shows most top-CQE origins sit in the *High (>1,600 m)* band, validating the terroir narrative buyers use in marketing.

---

### **Question 2 — Process-Driven Flavour Comparison**  
**What we ask**  
> *Inside those high-CQE origins, how do the three main processing methods — Washed, Natural, Honey — shift the six cupping attributes, and which method adds the most flavour without raising cost?*

#### **How we visualise it**  
* **Grouped bar chart** (one panel per selected origin).  
  * **X-axis** = six attributes (Aroma … Sweetness).  
  * **Three side-by-side bars** per attribute = Washed · Natural · Honey.  
  * **Y-axis** fixed 6 – 10 to magnify real differences.  
* **Dashboard filter** – selecting a country in the CQE scatter updates the grouped bars and the small process–price scatter simultaneously.

#### **Why it matters**  
* Lets buyers decide whether paying a *process premium* is justified.  
* Example from our workbook: Natural Ethiopians gain ≈ +0.8 Aroma & +0.6 Sweetness at the **same \$6 /kg** price, while Honey lots in Costa Rica raise cost but add little lift.

*Together, these questions move from a strategic view (which origins to source) to an operational one (which processes to request), demonstrating how merged price-quality-altitude data translates into actionable sourcing advice.*

---

## 7 | Visualization

Instead of providing screenshots here, our Dashboard visualization is on Tableau Public to preserve its interactive nature

# [Click Here To Explore Our Visualization ](https://public.tableau.com/shared/XRMHFSDX5?:display_count=n&:origin=viz_share_link)
