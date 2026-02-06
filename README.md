# Insurance & Natural Disasters Knowledge Graph

This project implements a Graph Data Science workflow to bridge the gap between **Physical Risk** (Natural Disasters) and **Financial Protection** (Insurance Penetration). By integrating disparate spatial, economic, and historical loss datasets into a unified Neo4j Knowledge Graph, we can perform advanced accumulation control, protection gap analysis, and risk community detection.


<p align="center">
  <img src="renderings/vulnerability_adjusted_exposure_BEL.png" alt="Economic Exposure"/>
  <br>
  <sub>Vulnerability-Adjusted Economic Exposure Map (30 arc-second resolution)</sub>
</p>

## Business Context

Natural catastrophes (NatCat) pose significant risks to insurers and reinsurers. To accurately manage these risks, it is necessary to understand the interaction between:

- **Economic Exposure:** The value of assets at risk.
- **Hazard Vulnerability:** The susceptibility of those assets to damage.
- **Financial Capacity:** The insurance market's ability to absorb losses.

This project demonstrates how Re/Insurers can use Neo4j as a data store to answer critical questions:

* **Accumulation Control:** Where are the "hotspots" of high economic value and high vulnerability?
* **Protection Gap:** Which regions have high expected losses but stressed insurance markets?
* **Risk Communities:** Which regions share similar multidimensional risk profiles?

## Data Sources

The Knowledge Graph is built using a Spatial ETL pipeline ingesting the following open datasets:

| Source | Dataset | Domain | Business Utility |
| --- | --- | --- | --- |
| **LitPop** | Global Exposure Data | Exposure Management | Proxy for Total Insurable Value (TIV) at 30 arc-second resolution (~1km). |
| **EIOPA** | Insurance Statistics | Market Intelligence | Aggregated Premiums, Claims & Expenses (Profitability & Capacity). |
| **DRMKC** | Risk Data Hub | Catastrophe Modelling | Vulnerability indicators and historical loss event catalogs (Flood, Storm, Earthquake). |

## Graph Data Model

The schema centers on **Location (Region)** as the unifying entity, connecting the Geo-Spatial, Risk, and Financial domains.

### Nodes & Relationships

* **`(:EconomicExposureCell)-[:LOCATED_IN]->(:Region)`**: Granular exposure units (LitPop) aggregated into NUTS3 administrative regions.
* **`(:Region)-[:HAS_VULNERABILITY]->(:Vulnerability)`**: Socio-economic vulnerability scores linked to regions.
* **`(:LossEvent)-[:IMPACTED_REGION]->(:Region)`**: Historical catastrophe events linked to the areas they impacted.
* **`(:Country)-[:REPORTED_FINANCIALS]->(:InsuranceMetric)`**: Financial KPIs (GWP, Claims, Ratios) linked to the top-level country node.

---

## Data Loading & Spatial ETL ([`loader.ipynb`](loader.ipynb))

The loader notebook handles the ingestion of high-resolution grid data and performs spatial joins to map economic exposure to the NUTS administrative hierarchy (NUTS1/2/3).

### Economic Exposure (TIV)

We load LitPop data to visualize Total Insurable Value density. Below is the economic exposure grid for Belgium.

### Aggregation

The granular cells are spatially joined with NUTS3 polygons to allow for regional reporting.

---

## Risk Analysis & Insights ([`analysis.ipynb`](analysis.ipynb))

The analysis notebook queries the graph to estimate risk, calculate protection gaps, and detect communities.

### National Insurance Market Capacity

We calculate the **Net Combined Ratio** from EIOPA data. A ratio > 100% indicates a stressed market (unprofitable underwriting), while < 100% indicates capacity to absorb new losses.

<div style="display: flex; justify-content: space-between;">
<img src="renderings/insurance_metrics_BEL.png" alt="Insurance Metrics Heatmap" width="48%"/>
<img src="renderings/insurance_gauge_BEL.png" alt="Market Capacity Gauge" width="48%"/>
</div>

### Empirical Loss Ratios

By analyzing historical `LossEvent` nodes, we calculate empirical loss ratios for Floods, Storms, and Earthquakes per region.

### Location-Specific Risk Assessment

We can query the graph for any specific coordinate (e.g., Verviers, BE) to estimate potential impacts. The model combines local exposure, regional vulnerability, and historical loss ratios to calculate **Conservative** vs. **Worst-Case** impact estimates.

### Regional Risk Mapping

Extending the point-based analysis, we generate risk maps for entire NUTS2 regions, highlighting areas with the highest potential financial impact from specific hazards.

---

## The Protection Gap Analysis

A key output of this project is the **Risk-Adjusted Loss Potential (RALP)**. This metric identifies regions where high Expected Annual Losses (EAL) coincide with a stressed insurance market.

### Dashboard: EAL vs RALP

This dashboard visualizes the "Risk Hotspots"—regions where disasters would cause economic shock that insurance cannot absorb.

<p align="center">
  <img src="renderings/ralp_heatmap_BEL.png" alt="RALP Heatnmap"/>
  <br>
  <sub>Risk Adjusted Loss Projections</sub>
</p>

![Risk Hotspots](renderings/ralp_heatmap_BEL.png)

### Bivariate Heatmap: The Sub-National Protection Gap

This map encodes two dimensions:

- **Expected Annual Loss (Horizontal):** Physical Risk.
- **Insurance Market Stress (Vertical):** Financial Capacity.

## Graph Data Science: Community Detection

Using the **Louvain Algorithm** via the Neo4j Graph Data Science (GDS) library, we project a graph based on vulnerability and economic exposure similarity (k-NN). This identifies natural clusters of regions with similar risk profiles, useful for portfolio segmentation.

## Getting Started

### Prerequisites

* **Neo4j Database** (AuraDB or Local)
* **Python 3.8+**
* **DRMKC API Token** (Required for vulnerability/loss data)

### Environment Setup

Create a `.env` file in the root directory:

```bash
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
ISO_A3_COUNTRY_CODE=BEL
DRMKC_TOKEN=your_token_here
USD_TO_EUR=0.92
```