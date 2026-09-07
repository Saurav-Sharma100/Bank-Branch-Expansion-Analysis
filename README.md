


# 🏦 Bank Branch Expansion Analysis

## 📌 Project Overview

Banks spend millions of dollars opening new branches, but selecting the wrong locations can lead to poor deposit growth and wasted capital.

This project addresses a real-world expansion scenario:

> **A regional bank has a $15 million expansion budget. Each new branch costs $2 million. The bank wants to open as many branches as possible while keeping new locations 10–25 km apart and maximizing new deposit capture.**

The goal was to build a **data-driven and reproducible process** that moves beyond simply ranking attractive markets. The final solution identifies high-potential ZIP codes, accounts for existing branch competition, enforces geographic spacing between proposed locations, and produces an actionable branch expansion plan.

The project combines:

- **Python** for data cleaning, EDA, market scoring, and analysis
- **Tableau** for strategic exploration of the national banking landscape
- **Streamlit** for interactive branch expansion simulation
- A **Greedy Optimization Algorithm** with **Haversine distance** to select spatially valid branch locations

---

## 📊 Dashboard Preview

<img width="1190" height="790" alt="Image" src="https://github.com/user-attachments/assets/2d626d51-93e1-40e1-8d1e-36dc871e4c11" />

---

## 🚀 Live Streamlit Application

The Streamlit application turns the analysis into an interactive decision tool where users can enter their own expansion constraints and generate a recommended branch plan.

**Live App:** [Open the Bank Branch Expansion Simulator](https://bankbranchapp.streamlit.app)

---

## 📋 Project Snapshot

| Category | Details |
|---|---|
| **Domain** | Banking / Financial Services |
| **Business Problem** | Bank Branch Expansion |
| **Data Source** | FDIC Summary of Deposits (SOD) 2025 |
| **Raw Records** | 76,120 branch records |
| **Cleaned Records** | 73,876 branches |
| **ZIP Markets Analyzed** | 17,971 |
| **Total Deposits Analyzed** | ~$18.10 Trillion |
| **Tools** | Python, Pandas, NumPy, Tableau, Streamlit, Plotly, Folium |
| **Core Metric** | Opportunity Score |
| **Optimization** | Greedy Branch Selection + Haversine Distance |
| **Final Output** | Ranked and spatially valid branch expansion plan |

---

# 🎯 Business Problem

The decision-maker for this project is a **strategy or growth analyst at a regional or national bank** who already has an approved expansion budget and needs to determine where that capital should be deployed.

The challenge is that the US contains thousands of potential banking markets. Choosing locations based on intuition or deposit size alone does not account for existing competition or geographic overlap.

The bank needs a process that can answer:

> **Which markets have strong deposit potential without already being saturated with branches, and which combination of those markets can actually be opened within the bank's budget and geographic constraints?**

Four business constraints drive the analysis:

| Constraint | Business Meaning |
|---|---|
| **Budget** | Total capital available for expansion |
| **Cost per Branch** | Capital required to open each location |
| **Minimum Distance** | Prevent new branches from being located too close together |
| **Target Region** | Allow expansion to focus on selected states or operate nationally |

The required output is therefore not just a spreadsheet of market scores, but a **ranked, spatially valid and executable branch plan**.

---

# 🗂️ Data Source

The analysis uses the **FDIC Summary of Deposits (SOD)** dataset.

Every FDIC-insured bank reports branch-level deposits annually, making the dataset suitable for comparing deposit potential and existing branch presence across markets.

The raw dataset contains one record per physical bank branch and includes:

| Field | Description |
|---|---|
| `CERT` | Unique FDIC bank identifier |
| `NAMEFULL` | Bank name |
| `ZIPBR` | Branch ZIP code |
| `STALPBR` | Branch state |
| `DEPSUMBR` | Branch deposits in thousands of USD |
| `SIMS_LATITUDE` | Branch latitude |
| `SIMS_LONGITUDE` | Branch longitude |
| `YEAR` | Reporting year |

---

# 🧹 Data Preparation & EDA

The raw FDIC dataset contained **76,120 records**. Python and Pandas were used to prepare the data before market-level analysis.

Key preparation steps included:

- Renaming FDIC fields into analysis-friendly column names
- Converting deposits from thousands into full USD
- Removing zero-deposit branches
- Standardizing ZIP codes to five digits
- Validating latitude and longitude
- Aggregating individual branches to ZIP and state level

After cleaning, **73,876 branches across 17,971 ZIP codes** remained for analysis.

### Dataset Summary

| Metric | Result |
|---|---:|
| Branches | 73,876 |
| Unique Banks | 3,895 |
| ZIP Codes | 17,971 |
| States / Territories | 58 |
| Total Deposits | ~$18.10T |
| Average Deposit per Branch | ~$245.05M |
| Median Deposit per Branch | ~$82.05M |

The large difference between average and median deposits reflects the strongly **right-skewed deposit distribution** observed during EDA: most branches hold much smaller balances while a relatively small number hold extremely large deposit pools.

---

# 🔍 Key Analytical Findings

### Deposit Concentration

Pareto analysis showed that:

> **The top 16% of ZIP codes hold approximately 80% of all deposits.**

This significantly narrows the expansion problem. Rather than treating every ZIP equally, the analysis can focus attention on the concentrated group of markets holding the majority of deposits.

### Deposit Potential vs. Branch Density

The analysis also showed that high deposits alone do not necessarily indicate a strong expansion opportunity.

Some ZIP codes contain large deposit pools but already have many existing branches. Others combine substantial deposits with comparatively limited branch coverage.

That gap between **deposit potential and existing branch density** became the basis of the Opportunity Score.

---

# 🧮 Opportunity Score

To compare markets fairly, deposits and branch counts were first normalized against their national ZIP-level averages.

### Normalized Deposit Base

```python
NORM_DEPOSIT_BASE = TOTAL_DEPOSITS / national_avg_TOTAL_DEPOSITS
```

This measures how large a ZIP's deposit pool is relative to the national average.

### Normalized Branch Density

```python
NORM_BRANCH_DENSITY = BRANCH_COUNT / national_avg_BRANCH_COUNT
```

This measures how saturated the ZIP is with existing bank branches.

### Opportunity Score

The two measures are combined as:

```python
OPPORTUNITY_SCORE = (
    0.6 * NORM_DEPOSIT_BASE
    - 0.4 * NORM_BRANCH_DENSITY
)
```

The score rewards **high deposit potential** while penalizing **high existing branch density**.

```text
Positive Score  → Underserved, high-value market
Negative Score  → Overserved relative to its deposit base
```

Deposit potential receives the larger weight because the primary objective of the expansion strategy is deposit capture.

---

# 💰 Expected Deposit Capture

The Opportunity Score is converted into an estimated capture percentage:

```python
CAPTURE_PCT = clamp(
    0.02 * OPPORTUNITY_SCORE,
    0.01,
    0.10
)
```

The estimated capture rate is constrained between **1% and 10%**.

Expected deposit capture is then calculated as:

```python
EXPECTED_CAPTURE_USD = TOTAL_DEPOSITS * CAPTURE_PCT
```

This gives each ZIP both an analytical ranking and an estimated dollar opportunity.

---

# 🏆 Top Expansion Opportunities

The scoring model ranks all **17,971 ZIP markets** based on their deposit potential relative to existing branch density.

The top five markets from the analysis were:

| ZIP | State | Total Deposits | Branches | Opportunity Score | Expected Capture |
|---|---|---:|---:|---:|---:|
| **10017** | NY | $744.59B | 20 | **441.55** | $74.46B |
| **57108** | SD | $531.39B | 31 | **313.49** | $53.14B |
| **84111** | UT | $485.48B | 22 | **287.02** | $48.55B |
| **28202** | NC | $397.80B | 10 | **235.96** | $39.78B |
| **57105** | SD | $344.74B | 6 | **204.75** | $34.47B |

However, simply selecting the highest-ranked ZIP codes would not solve the full business problem.

Two highly ranked ZIPs could be located within the same area. Opening both could create geographic overlap and cannibalization.

This is where the optimization stage becomes necessary.

---

# ⚙️ Greedy Branch Selection Algorithm

The Streamlit simulator uses a **Greedy Branch Selection Algorithm** to turn the ZIP rankings into a practical branch portfolio.

The process is:

1. **Filter** candidates based on region, score and capture requirements.
2. **Sort** eligible ZIPs by Opportunity Score from highest to lowest.
3. **Seed** the portfolio with the highest-ranked ZIP.
4. **Evaluate** each remaining candidate against locations already selected.
5. **Add or skip** the ZIP depending on whether it satisfies the minimum-distance requirement.
6. **Stop** when the budget is exhausted or no valid candidates remain.

This prevents the model from simply selecting several high-scoring locations clustered within the same area.

---

## 🌎 Haversine Distance

The optimizer uses the **Haversine formula** to calculate great-circle distance between latitude/longitude coordinates.

For every candidate ZIP, its distance from all previously selected branches is calculated.

```text
Candidate ZIP
      ↓
Distance to all selected locations
      ↓
At least minimum distance away?
      ↓
 YES → Select        NO → Skip
```

The implementation is vectorized with NumPy so one candidate can be compared against all already-selected branches simultaneously.

This allows geographic separation to become part of the actual branch-selection logic rather than being checked manually after locations have already been chosen.

---

# 📊 Tableau Dashboard

The Tableau dashboard provides the **strategic exploration layer** of the project.

It answers:

> **Where are the strongest expansion opportunities across the national banking landscape?**

The dashboard contains four views.

### 🗺️ National Opportunity Map

A national ZIP-level map where deposit size and Opportunity Score make high-potential markets geographically visible.

### 📈 Top 20 ZIP Rankings

Ranks the strongest ZIP-level expansion opportunities by Opportunity Score.

### 📉 Deposit Pareto

Shows the concentration of deposits across ZIP codes and highlights the finding that approximately **16% of ZIPs account for 80% of deposits**.

### 🏦 State Summary

Compares state-level **total deposits and branch counts**, providing a broader view of market size versus existing banking infrastructure.

---

# 🖥️ Streamlit Branch Expansion Simulator

<img width="1913" height="919" alt="Image" src="https://github.com/user-attachments/assets/1df04ef6-37b3-4033-8b67-60e6fca8f32f" />

---

The Streamlit application takes the project from **market analysis to branch planning**.

While Tableau helps the user explore where opportunities exist, Streamlit answers:

> **Given my budget and constraints, what branch locations should I actually select?**

Users can configure:

- Target states
- Total expansion budget
- Cost per branch
- Minimum distance between branches
- Minimum expected capture per branch
- Maximum branch override

The application then runs the optimizer and returns:

- Number of branches selected
- Total expected deposit capture
- Budget utilization
- Average capture per branch
- Recommended ZIP-level branch plan
- Geographic map of selected locations
- Downloadable branch plan CSV

---

## 🔄 Scenario Comparison

The application also compares two expansion approaches:

| | Conservative | Aggressive |
|---|---|---|
| **Candidate Pool** | Top 25% by Opportunity Score | All positive-score ZIPs |
| **Minimum Distance** | 2× selected distance | 0.5× selected distance |
| **Result** | Fewer, higher-confidence locations | More branches with tighter spacing |

The comparison includes branch count, expected capture, average capture, average Opportunity Score, and budget utilization.

This gives the decision-maker the ability to compare different expansion strategies rather than relying on one fixed recommendation.

---

# 🔗 Tableau vs. Streamlit

The two analytical products serve different stages of the same business decision.

| Tableau Dashboard | Streamlit Simulator |
|---|---|
| **Audience:** Executives and strategy analysts | **Audience:** Strategy team with budget and constraints |
| **Question:** Where are the best opportunities? | **Question:** What is the branch plan? |
| Filter, explore and compare markets | Enter parameters and run simulation |
| Visual market landscape | Recommended branch locations |
| Early-stage exploration | Decision-making and execution |

Together, they move the project from **strategic market exploration to an actionable expansion plan**.

---

# 🛠️ Tools & Technologies

| Tool | Use |
|---|---|
| **Python** | Data preparation, EDA and analytical logic |
| **Pandas** | Cleaning, transformation and aggregation |
| **NumPy** | Vectorized Haversine distance calculations |
| **Matplotlib / Plotly** | Exploratory and interactive visualizations |
| **Tableau** | National opportunity dashboard |
| **Streamlit** | Interactive branch expansion simulator |
| **Folium** | Geographic opportunity and simulation maps |
| **FDIC SOD** | Branch-level deposit and geographic data |

---

# 📦 Project Deliverables

The completed project produces:

- **EDA Notebook** — data cleaning, exploratory analysis, market aggregation and scoring
- **`zip_metrics.csv`** — ZIP-level opportunity metrics for 17,971 markets
- **`state_metrics.csv`** — state-level deposit and branch summaries
- **`branch_metrics.csv`** — cleaned branch-level dataset
- **Tableau Dashboard** — national opportunity exploration
- **Streamlit Application** — interactive expansion simulator
- **Branch Plan CSV** — downloadable output from the optimizer

---

# 🏁 Conclusion

This project was built to answer a specific banking decision: **where should a bank deploy a limited branch-expansion budget to capture the most deposits while avoiding geographic overlap?**

Starting with **76,120 raw FDIC branch records**, the analysis cleaned and aggregated the data into **17,971 ZIP-level markets**, identified that approximately **16% of ZIP codes hold 80% of deposits**, and created an Opportunity Score that evaluates deposit potential against existing branch density.

The Tableau dashboard provides the strategic view of where attractive markets exist, while the Streamlit simulator applies the bank's actual budget, branch cost, regional filters and minimum-distance requirement to generate a spatially valid branch plan.

The final result is therefore not just an analysis of the US banking landscape, but a process that moves from **raw FDIC data → market opportunity → business constraints → recommended branch locations**.
