# 🏦 Bank Branch Expansion Simulator


An end-to-end analytics and optimization project that uses **FDIC Summary of Deposits (SOD)** data to identify high-potential locations for new bank branches while respecting **budget and geographic distance constraints**.

The project combines **Python, Pandas, NumPy, Haversine distance calculations, Streamlit, and Tableau** to transform raw government banking data into an actionable branch expansion plan.

---

## 🚀 Live Application

### Streamlit Simulator

👉 **[Launch the Bank Branch Expansion Simulator](https://bankbranchexpansionsimulator-pne8xmbxz6z2tcu9e4xqpk.streamlit.app/)**

The interactive application allows users to:

* Upload FDIC Summary of Deposits data
* Select target states
* Set an expansion budget
* Set branch opening cost
* Define minimum distance between new branches
* Set minimum expected deposit capture
* Set a maximum branch limit
* Run branch expansion simulations
* Compare conservative and aggressive scenarios
* View recommended locations on an interactive map
* Download the resulting branch plan as CSV

---

## 📂 Project Repository

👉 **[Bank Branch Expansion Simulator — GitHub](https://github.com/Sahil-Sharma-603/Bank_Branch_Expansion_Simulator)**

---

# 📌 Project Overview

Banks invest significant amounts of capital when expanding their physical branch networks. Selecting the wrong locations can result in:

* Low deposit growth
* Excessive competition
* Branch cannibalization
* Poor capital utilization
* Underperforming locations

This project addresses the problem as a **constrained geographic optimization problem**.

A hypothetical regional bank has:

* **$15 million expansion budget**
* **$2 million cost per new branch**
* A requirement that new branches remain **10–25 km apart**
* A goal of maximizing potential new deposit capture

The simulator analyzes thousands of US ZIP codes and produces a spatially valid branch expansion plan.

---

# 🎯 Project Objective

The goal is to answer two key business questions:

> **Where should a bank consider opening new branches?**

and

> **Given a fixed budget and geographic constraints, which combination of locations provides the highest modeled deposit opportunity?**

The final output is not simply a ranking of ZIP codes.

Instead, the optimizer produces an **executable branch plan** that respects:

* Available budget
* Branch opening cost
* Minimum geographic distance
* Target regions
* Deposit opportunity
* Expected deposit capture

---

# 🏛️ Why FDIC Summary of Deposits?

The project uses the **FDIC Summary of Deposits (SOD)** dataset.

The FDIC requires FDIC-insured institutions to report deposit information for their domestic offices annually. The resulting dataset provides branch-level information that can be aggregated and analyzed geographically.

### Dataset characteristics

| Feature                | Description              |
| ---------------------- | ------------------------ |
| Source                 | FDIC Summary of Deposits |
| Coverage               | United States            |
| Frequency              | Annual                   |
| Historical data        | 1994+                    |
| Branch records         | ~85K+ annually           |
| Geographic information | ZIP, latitude, longitude |
| Deposit information    | Branch-level deposits    |
| Cost                   | Free                     |
| API key                | Not required             |

### Official FDIC Source

👉 **[FDIC Summary of Deposits — Bulk Data Downloads](https://banks.data.fdic.gov/bankfind-suite/SOD/customDownload)**

The dataset is publicly available and can be used to reproduce the analysis.

---

# 📊 Key Dataset Fields

The primary fields used in this project are:

| Field            | Type    | Description                         |
| ---------------- | ------- | ----------------------------------- |
| `CERT`           | Integer | FDIC Certificate / bank identifier  |
| `ZIPBR`          | String  | Five-digit branch ZIP code          |
| `DEPSUMBR`       | Float   | Branch deposits in thousands of USD |
| `SIMS_Latitude`  | Float   | Branch latitude                     |
| `SIMS_Longitude` | Float   | Branch longitude                    |
| `STALPBR`        | String  | Two-letter state abbreviation       |
| `YEAR`           | Integer | SOD reporting year                  |

Deposit values are converted from thousands of USD into full USD values during data cleaning.

---

# 🧠 Business Problem

The branch expansion problem can be represented as:

### Given

* Total expansion budget
* Cost per branch
* Geographic target area
* Minimum distance between branches
* Expected deposit capture requirements

### Determine

A set of branch locations that maximizes modeled deposit opportunity while satisfying all constraints.

### Example

```text
Total Budget       = $15M
Cost / Branch      = $2M
Minimum Distance   = 10 km
Target Region      = Selected states
```

The theoretical budget limit is:

```text
$15M / $2M = 7 branches
```

The optimizer may select fewer branches if geographic or opportunity constraints prevent additional valid selections.

---

# 🔄 End-to-End Workflow

```text
FDIC SOD Data
      │
      ▼
┌───────────────┐
│    Ingest      │
│ data_loader.py │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│     Clean     │
│ Normalize data│
└───────┬───────┘
        │
        ▼
┌────────────────┐
│     Analyse     │
│ ZIP-level       │
│ metrics         │
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Opportunity     │
│ Score           │
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Greedy          │
│ Optimizer       │
└────────┬───────┘
         │
         ▼
┌────────────────┐
│ Streamlit       │
│ Simulator       │
└────────┬───────┘
         │
         ├──────────────► Branch Plan CSV
         │
         └──────────────► Interactive Map
         
FDIC Data ─────────────► Tableau Dashboard
```

---

# 🧹 Data Preparation

The raw FDIC SOD dataset contains approximately 85,000 branch records and can be several gigabytes in size depending on the reporting year.

The data preparation pipeline performs the following steps:

### 1. Load raw SOD data

Annual FDIC SOD CSV data is loaded into the processing pipeline.

### 2. Normalize column names

Column names are standardized across different SOD data vintages.

### 3. Convert deposits

FDIC deposits are reported in thousands of USD.

They are converted using:

```python
DEPOSITS_USD = DEPSUMBR * 1000
```

### 4. Remove zero-deposit records

Zero-deposit branches are removed because they may represent ATMs, shells, or locations that do not contribute to the deposit opportunity analysis.

### 5. Validate ZIP codes

ZIP codes are converted to five-character strings and zero-padded where necessary.

Example:

```text
1234 → 01234
```

### 6. Validate coordinates

Latitude and longitude values are checked for validity before geographic calculations.

---

# 📈 Exploratory Data Analysis

The EDA investigates several characteristics of the US banking landscape.

## Deposit Distribution

Branch deposits are highly right-skewed.

Most branches have relatively modest deposit balances while a smaller number of branches hold very large deposit pools.

This means that the **mean deposit balance can be substantially higher than the median**.

---

## Deposit Concentration

Deposits are geographically concentrated.

The analysis includes a Pareto-style view of ZIP-level deposits to identify how concentrated total deposits are across US markets.

This helps identify ZIP codes with disproportionately large deposit pools.

---

## Geographic Concentration

The analysis examines deposits and branch counts by state.

This helps distinguish between:

* States with large overall deposit pools
* States with high branch density
* Markets where deposit opportunity may be relatively high compared with existing branch presence

---

## Deposit Density vs Branch Density

A high number of existing branches does not necessarily mean a ZIP code has the highest deposit opportunity.

The project therefore compares:

```text
Deposit Base
      vs
Existing Branch Density
```

This forms the foundation for the Opportunity Score.

---

# 📐 Opportunity Score

Each ZIP code receives an Opportunity Score based on its deposit base and existing branch density.

The scoring model is intentionally transparent and uses two primary signals.

---

## Step 1 — Normalize Deposit Base

```text
Normalized Deposit Base
=
ZIP Total Deposits
/
National Average ZIP Deposits
```

A value greater than 1 indicates that the ZIP has a deposit pool above the national ZIP-level average.

---

## Step 2 — Normalize Branch Density

```text
Normalized Branch Density
=
ZIP Branch Count
/
National Average Branch Count
```

A higher value indicates greater existing branch presence.

---

## Step 3 — Calculate Opportunity Score

```text
Opportunity Score
=
(0.6 × Normalized Deposit Base)
-
(0.4 × Normalized Branch Density)
```

The model therefore rewards ZIP codes with:

* Large deposit pools
* Relatively lower existing branch density

---

# 💰 Expected Deposit Capture

The project converts Opportunity Score into an estimated deposit capture percentage.

```text
Capture %
=
CLAMP(
    0.02 × Opportunity Score,
    0.01,
    0.10
)
```

This limits modeled capture to:

```text
Minimum = 1%
Maximum = 10%
```

Expected captured deposits are then calculated as:

```text
Expected Capture
=
ZIP Total Deposits × Capture %
```

### Example

If a ZIP has:

```text
Total Deposits = $100M
Capture Rate   = 6%
```

Then:

```text
Expected Capture = $100M × 6%
                 = $6M
```

> **Important:** Expected Capture is a modeled analytical estimate, not a forecast or guarantee of actual future deposits. The project uses it as a consistent opportunity metric for comparing candidate locations.

---

# 🧮 Branch Expansion Optimizer

Simply selecting the highest-scoring ZIP codes would not work.

For example:

```text
ZIP A → Opportunity Score = 8.5
ZIP B → Opportunity Score = 8.3
ZIP C → Opportunity Score = 8.1
```

If all three ZIPs are located within a few kilometers of one another, opening branches at all three locations could create substantial geographic overlap.

The optimizer therefore applies a **minimum distance constraint**.

---

# 📍 Greedy Optimization Algorithm

The project uses a greedy selection strategy.

### Step 1 — Filter

Candidates are filtered according to:

* Target state / region
* Opportunity Score
* Expected Capture threshold
* Other user-defined constraints

### Step 2 — Sort

Candidates are sorted by:

```text
Opportunity Score DESC
```

### Step 3 — Seed

The highest-ranked candidate is selected as the first branch.

### Step 4 — Iterate

For every subsequent candidate:

```text
Calculate distance
to every selected branch
```

### Step 5 — Distance Check

If:

```text
Distance >= Minimum Distance
```

the candidate is selected.

Otherwise it is skipped.

### Step 6 — Stop

The algorithm stops when:

* The budget is exhausted, or
* The maximum branch count is reached, or
* No remaining candidate satisfies the spatial constraints

---

# 🌎 Haversine Distance

Geographic distance is calculated using the Haversine formula.

The formula calculates the great-circle distance between two latitude/longitude coordinates.

Conceptually:

```text
Latitude + Longitude
        ↓
Haversine Distance
        ↓
Distance in Kilometers
```

This allows the optimizer to enforce constraints such as:

```text
Minimum Distance = 10 km
```

between every pair of selected branch locations.

---

# 💵 Budget Constraint

The optimizer also enforces the available expansion budget.

For example:

```text
Budget             = $15M
Cost per Branch    = $2M
```

Maximum possible branches:

```text
floor($15M / $2M) = 7
```

The actual number selected may be lower depending on the spatial and opportunity constraints.

---

# 🖥️ Streamlit Application

The Streamlit application turns the analytical pipeline into an interactive branch expansion simulator.

### Application inputs

Users can configure:

* FDIC SOD CSV
* Target states
* Total expansion budget
* Cost per branch
* Minimum distance between branches
* Minimum expected capture
* Maximum number of branches

---

# 🗺️ Application Features

## EDA Overview

The application provides:

* KPI cards
* State-level deposit summaries
* Deposit distribution
* Pareto analysis
* Top ZIP codes
* Branch-level information

---

## Opportunity Map

A national interactive map displays ZIP-level opportunities.

Map characteristics include:

* Bubble size → Total Deposits
* Bubble colour → Opportunity Score
* ZIP-level geographic information

Users can explore individual markets directly from the map.

---

## Simulation Results

After running a simulation, the application displays:

### Portfolio KPIs

Examples include:

* Number of recommended branches
* Budget utilized
* Total expected deposit capture
* Average Opportunity Score

### Branch Plan

The recommended branch locations are displayed in a sortable table.

Example fields:

```text
ZIP
City
State
Opportunity Score
Expected Capture
Distance to Nearest Selected Branch
```

The resulting branch plan can be downloaded as CSV.

---

# 🔄 Scenario Comparison

The application supports scenario analysis.

Two example scenarios are:

### Conservative Scenario

* Higher Opportunity Score threshold
* Wider geographic spacing
* More selective candidate pool

### Aggressive Scenario

* Broader candidate pool
* Tighter geographic spacing
* More locations considered

This allows users to examine how changes in assumptions affect the resulting branch portfolio.

---

# ⚡ Performance and Caching

The raw SOD dataset can be approximately **1.5 GB or larger**, depending on the source year and file format.

Loading and processing the entire dataset every time a user changes a Streamlit input would make the application slow.

The application therefore uses Streamlit caching.

### Processing architecture

```text
First Load
    │
    ▼
Raw FDIC CSV
    │
    ▼
Cleaned DataFrame
    │
    ▼
Cached Data
```

Then:

```text
Cached ZIP Metrics
        │
        ▼
Optimizer
        │
        ▼
Simulation Results
```

This separates expensive data preparation from lightweight simulation runs.

---

# 📊 Tableau Dashboard

The Tableau dashboard is designed for strategic exploration, while the Streamlit application is designed for actionable simulation.

### Tableau audience

* Executives
* Strategy teams
* Market planning teams
* Business analysts

### Key questions

```text
Where are the largest opportunities?

Which markets have large deposits?

Where is branch density relatively high or low?

How concentrated are US deposits?
```

### Planned Dashboard Views

#### 1. National Opportunity Map

ZIP-level bubble map showing:

* Total deposits
* Opportunity Score
* Geographic distribution

#### 2. Top ZIP Rankings

Top ZIP codes ranked by Opportunity Score.

#### 3. Deposit Concentration

Pareto-style visualization showing cumulative deposit share across ZIP markets.

#### 4. State Summary

Comparison of:

* Total deposits
* Branch count
* Geographic distribution



---

# 📁 Project Deliverables

The project produces the following deliverables:

```text
├── EDA Notebook
│   └── 01_eda_analysis.ipynb
│
├── Processed Data
│   ├── zip_metrics.csv
│   ├── state_metrics.csv
│   └── branches.csv
│
├── Streamlit Application
│   └── Interactive branch expansion simulator
│
├── Tableau Dashboard
│   └── Strategic opportunity exploration
│
├── Branch Plan
│   └── branch_expansion_plan.csv
│
└── README.md
```



# 🛠️ Technology Stack

| Technology                          | Purpose                            |
| ----------------------------------- | ---------------------------------- |
| Python                              | Data processing and optimization   |
| Pandas                              | Data cleaning and aggregation      |
| NumPy                               | Numerical calculations             |
| Haversine / Geographic calculations | Branch distance constraints        |
| Streamlit                           | Interactive simulation application |
| Folium                              | Interactive geographic maps        |
| Tableau                             | Strategic visualization            |
| FDIC SOD                            | Government banking data            |

---


# 🔗 Project Links

| Resource              | Link                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------ |
| 💻 GitHub Repository  | [Bank Branch Expansion Simulator](https://github.com/Sahil-Sharma-603/Bank_Branch_Expansion_Simulator) |
| 🚀 Live Streamlit App | [Launch Simulator](https://bankbranchexpansionsimulator-pne8xmbxz6z2tcu9e4xqpk.streamlit.app/)         |
| 🏛️ FDIC SOD Dataset  | [FDIC Summary of Deposits](https://banks.data.fdic.gov/bankfind-suite/SOD/customDownload)              |
| 📊 Tableau Dashboard  | **Add Tableau Public link**                                                                            |

---


# 🎓 Project Learning Outcomes

This project demonstrates an end-to-end analytics engineering workflow:

### Data Engineering

* Working with large public datasets
* Data cleaning and normalization
* Handling geographic data
* Building reusable data-processing functions

### Analytics

* Exploratory data analysis
* Distribution analysis
* Geographic aggregation
* Deposit concentration analysis
* Feature normalization
* Opportunity scoring

### Optimization

* Constraint-based candidate selection
* Greedy optimization
* Geographic distance calculations
* Budget-constrained portfolio construction

### Application Development

* Streamlit application development
* Interactive maps
* User-controlled simulation parameters
* Caching for performance
* Downloadable analytical outputs

### Business Communication

* Translating analytical outputs into business decisions
* Separating exploratory analysis from actionable recommendations
* Presenting results through both Tableau and Streamlit

---

# 📌 Final Outcome

The Bank Branch Expansion Simulator transforms publicly available FDIC banking data into a reproducible decision-support workflow:

```text
Raw Government Data
        ↓
Data Cleaning
        ↓
ZIP-Level Metrics
        ↓
Opportunity Scoring
        ↓
Spatial Optimization
        ↓
Interactive Simulation
        ↓
Recommended Branch Plan
```

The result is an analytical product that allows a strategy team to move from **"Where are the opportunities?"** to **"What branch portfolio satisfies our budget and geographic constraints?"**

---

## 👤 Author

**Sahil Sharma**

### Project

**Bank Branch Expansion Simulator**


---

## 📜 Data Source

Federal Deposit Insurance Corporation (FDIC)

**Summary of Deposits (SOD)**

The FDIC Summary of Deposits dataset is publicly available through the FDIC BankFind Suite.

[FDIC BankFind Suite — Summary of Deposits](https://banks.data.fdic.gov/bankfind-suite/SOD/customDownload)
