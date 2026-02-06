# Insurance & Natural Disasters Knowledge Graph

This project implements a [Graph Data Science](https://neo4j.com/product/graph-data-science/) workflow to bridge the gap between **Physical Risk** (Natural Disasters) and **Financial Protection** (Insurance Penetration). By integrating disparate spatial, economic, and historical loss datasets into a unified Neo4j Knowledge Graph, we can perform advanced accumulation control, protection gap analysis, and risk community detection.

It intends to show prospective customers in the Insurance and Risk Management space how Neo4j can act as a geo aware datastore, while enriching internal policy data with external hazard landscapes to reveal hidden correlations and spatial dependencies that traditional relational databases miss.

The resulting Knowledge Graph serves as a dynamic risk engine. Instead of viewing a property as a static row in a table, we model it as a node connected to a web of environmental and economic factors. This allows insurers to move beyond simple "zip code" underwriting to hyper-local risk assessment.

<p align="center">
  <img src="renderings/vulnerability_adjusted_exposure_BEL.png" alt="Economic Exposure"/>
  <br>
  <sub>Vulnerability-Adjusted Economic Exposure Map (30 arc-second resolution)</sub>
</p>


## Business Context

Natural catastrophes (NatCat) pose significant risks to insurers and reinsurers. Effectively managing these risks requires a deep understanding of how economic exposure, hazard vulnerability, and financial capacity interact. Economic exposure refers to the value of assets at risk, while hazard vulnerability describes how susceptible those assets are to damage from natural events. Financial capacity, on the other hand, is the insurance market's ability to absorb losses when disasters strike.

This project demonstrates how Re/Insurers can leverage Neo4j as a data store to answer critical business questions. For example, it enables the identification of accumulation hotspots-areas where high economic value coincides with high vulnerability. It also helps to reveal protection gaps by highlighting regions with high expected losses but stressed insurance markets. Finally, we demonstrate how unsupervised similarity detection algorithms available in Neo4j can be used to identify **risk communities**, allowing users to find regions that share similar multidimensional risk profiles.

## Data Sources

The Knowledge Graph is built using a Spatial ETL pipeline ingesting the following open datasets:

| Source | Dataset | Domain | Business Utility |
| --- | --- | --- | --- |
| [**LitPop**](https://www.research-collection.ethz.ch/entities/researchdata/12dcfc4f-9d03-463a-8d6b-76c0dc73cdc8) | Global Exposure Data | Exposure Management | Proxy for Total Insurable Value (TIV) at 30 arc-second resolution (~1km). |
| [**EIOPA**](https://www.eiopa.europa.eu/tools-and-data/insurance-statistics_en#premiums-claims-and-expenses) | Insurance Statistics | Market Intelligence | Aggregated Premiums, Claims & Expenses (Profitability & Capacity). |
| [**DRMKC**](https://drmkc.jrc.ec.europa.eu/risk-data-hub) | Risk Data Hub | Catastrophe Modelling | Vulnerability indicators and historical loss event catalogs (Flood, Storm, Earthquake). |

## Graph Data Model

The schema centers on **Location (Region)** as the unifying entity, connecting the Geo-Spatial, Risk, and Financial domains.

### Nodes & Relationships

* **`(:EconomicExposureCell)-[:LOCATED_IN]->(:Region)`**: Granular exposure units (LitPop) aggregated into NUTS3 administrative regions.
* **`(:Region)-[:HAS_VULNERABILITY]->(:Vulnerability)`**: Socio-economic vulnerability scores linked to regions.
* **`(:LossEvent)-[:IMPACTED_REGION]->(:Region)`**: Historical catastrophe events linked to the areas they impacted.
* **`(:Country)-[:REPORTED_FINANCIALS]->(:InsuranceMetric)`**: Financial KPIs (GWP, Claims, Ratios) linked to the top-level country node.

<p align="center">
  <img src="renderings/schema_graph.png" alt="Graph Schema"/>
  <br>
  <sub>The Graph Schema</sub>
</p>

## Data Loading & Spatial ETL ([`loader.ipynb`](loader.ipynb))

The loader notebook handles the ingestion of high-resolution grid data and performs spatial joins to map economic exposure to the [NUTS](https://ec.europa.eu/eurostat/web/nuts) administrative hierarchy (NUTS1/2/3).

### Economic Exposure (TIV)

We load LitPop data to visualize Total Insurable Value density. Below is the economic exposure grid for Belgium.

### Aggregation

The granular cells are spatially joined with NUTS3 polygons to allow for regional reporting.

## Risk Analysis & Insights ([`analysis.ipynb`](analysis.ipynb))

The analysis notebook queries the graph to estimate risk, calculate protection gaps, and detect communities.

### National Insurance Market Capacity

We calculate the **Net Combined Ratio** from EIOPA data. A ratio > 100% indicates a stressed market (unprofitable underwriting), while < 100% indicates capacity to absorb new losses.

<p align="center">
  <img src="renderings/insurance_metrics_BEL.png" alt="Insurance Metrics"/>
  <br>
  <sub>Risk Adjusted Loss Projections</sub>
</p>

### Empirical Loss Ratios

By analyzing historical `LossEvent` nodes, we calculate empirical loss ratios for Floods, Storms, and Earthquakes per region.

### Location-Specific Risk Assessment

We can query the graph for any specific coordinate (e.g., Verviers in Belgium, which is a flood prone region) to estimate potential impacts. The model combines local exposure, regional vulnerability, and historical loss ratios to calculate **Conservative** vs. **Worst-Case** impact estimates.

<p align="center">
  <img src="renderings/location_risk_BE335_Flood.png" alt="Location Specific Risk Dashboard"/>
  <br>
  <sub>Location Specific Risk Dashboard</sub>
</p>

### Regional Risk Mapping

Extending the point-based analysis, we generate risk maps for entire NUTS2 regions, highlighting areas with the highest potential financial impact from specific hazards.

<p align="center">
  <img src="renderings/nuts3_risk_map_Flood_BEL.png" alt="Location Specific Risk Map"/>
  <br>
  <sub>Location Specific Risk Map</sub>
</p>

## The Protection Gap Analysis

A key output of this project is the **Risk-Adjusted Loss Potential (RALP)**. This metric identifies regions where high Expected Annual Losses (EAL) coincide with a stressed insurance market.

### Dashboard: EAL vs RALP

This dashboard visualizes the "Risk Hotspots"—regions where disasters would cause economic shock that insurance cannot absorb.

<p align="center">
  <img src="renderings/ralp_heatmap_BEL.png" alt="RALP Heatmap"/>
  <br>
  <sub>Risk Adjusted Loss Projections</sub>
</p>

### Bivariate Heatmap: The Sub-National Protection Gap

This map encodes two dimensions:

- Expected Annual Loss (Horizontal): Physical Risk.
- Insurance Market Stress (Vertical): Financial Capacity.

<p align="center">
  <img src="renderings/protection_gap_bivariate_BEL.png" alt="Bivariate Protection Gap Heatmap"/>
  <br>
  <sub>Bivariate Protection Gap Heatmap</sub>
</p>

## Graph Data Science: Community Detection

Using the **Louvain Algorithm** via the Neo4j Graph Data Science (GDS) library, we project a graph based on vulnerability and economic exposure similarity (k-NN). This identifies natural clusters of regions with similar risk profiles, useful for portfolio segmentation.

<p align="center">
  <img src="renderings/risk_communities_BEL.png" alt="Similar Risk Communities in BEL"/>
  <br>
  <sub>Similar Risk Communities in BEL</sub>
</p>

## Potential Architecture for Productionization

In a production enterprise environment, we move away from siloed data lakes toward an operational graph that serves real-time underwriting and strategic portfolio management.

The following diagram illustrates how this solution could fit into a modern insurance technology stack:

```mermaid
flowchart TB
    subgraph Sources ["Data Sources"]
        direction TB
        Ext_Haz[("External Hazard Data<br/>(DRMKC)")]
        Ext_Exp[("Global Exposure<br/>(LitPop)")]
        Ext_Fin[("Market Stats<br/>(EIOPA)")]
        Int_Pol[("Internal Policy Data<br/>(PAS / Claims)")]
    end

    subgraph Ingestion ["Ingestion & Spatial ETL"]
        Py_Loader["Python Spatial Loader<br/>(GeoPandas / Sedona)"]
        Clean["Data Cleaning &<br/>Coordinate Transform"]
    end

    subgraph KnowledgeGraph ["Risk Knowledge Graph (Neo4j)"]
        direction TB
        N_DB[("Neo4j Database")]
        
        subgraph GDS ["Graph Data Science"]
            Algo1["Community Detection<br/>(Louvain)"]
            Algo2["Similarity Search<br/>(k-NN)"]
        end
    end

    subgraph AppLayer ["Application & Consumption"]
        UW_Bench["Underwriting Workbench<br/>(Real-time Risk Check)"]
        Dash["Portfolio Dashboard<br/>(Accumulation & RALP)"]
        Reins["Reinsurance Reporting<br/>(Aggregated Exposure)"]
    end

    %% Relationships
    Ext_Haz --> Py_Loader
    Ext_Exp --> Py_Loader
    Ext_Fin --> Py_Loader
    Int_Pol --> Py_Loader
    
    Py_Loader --> Clean
    Clean -->|Construct Nodes & Rels| N_DB
    
    N_DB <--> Algo1
    N_DB <--> Algo2
    
    N_DB -->|Query: Risk Score| UW_Bench
    N_DB -->|Query: Aggregations| Dash
    N_DB -->|Query: Treaty Data| Reins
    
    %% Styling
    classDef source fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef process fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef db fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef app fill:#f3e5f5,stroke:#4a148c,stroke-width:2px;
    
    class Ext_Haz,Ext_Exp,Ext_Fin,Int_Pol source;
    class Py_Loader,Clean process;
    class N_DB,Algo1,Algo2 db;
    class UW_Bench,Dash,Reins app;
```

#### Core Components

**Data Ingestion Layer (Spatial ETL)**

- External Feeds: Connects to APIs (such as DRMKC in our example) and bulk data sources (LitPop, EIOPA) for hazard and economic data.
- Internal Data: Ingests policy administration systems (PAS) and claims databases.
- Geospatial Processing: Uses Python (GeoPandas) or Apache Sedona to handle coordinate transformations and spatial joins before graph loading.


**The Risk Knowledge Graph (Neo4j)**

- The "Golden Record" for Risk: Acts as the central registry linking Policyholders, Physical Assets, Hazards, and Financial Capacity.
- Graph Data Science (GDS): Runs in-database algorithms (Louvain, k-NN) to compute similarity scores and detect risk communities without moving data.


**Application Layer**

- Underwriting Workbench: Queries the graph in real-time (via GraphQL or Bolt) to fetch "risk proximity" scores for a specific address during the quoting process.
- Portfolio Dashboard: Visualizes accumulation risk and protection gaps (as seen in the RALP analysis).
- Reinsurance Reporting: Aggregates graph data to report total exposure by hazard zones for treaty negotiations.

#### Enterprise Integration Flow

- Quote Request: A customer applies for property insurance.
- Graph Query: The application queries the Knowledge Graph for the property's location.
- Risk Enrichment: The graph traverses relationships to identify:
  - Direct Hazard: Is the property in a flood zone?
- Accumulation: Do we already insure 50 neighbors in this specific flood community?
- Market Health: Is the local combined ratio already >100%?


- Decision: The underwriter receives a composite "Risk Score" and a "Decline/Refer" recommendation based on graph metrics.

## Getting Started with these Notebooks

### Prerequisites

- Neo4j Database (AuraDB or Local)
- Python 3.8+
- DRMKC API Token (Required for vulnerability/loss data)

You can create a free Neo4j AuraDB instance [here](https://neo4j.com/cloud/aura/). Make sure to note down your connection URI, username, and password for the environment setup.

To register for a DRMKC API token, visit the [DRMKC Risk Data Hub API documentation](https://drmkc.jrc.ec.europa.eu/risk-data-hub-api/docs/) and follow their instructions to obtain access.

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

You can set the target country for the analysis by changing the `ISO_A3_COUNTRY_CODE` variable (e.g., "FRA" for France, "ITA" for Italy). Before running the notebooks, ensure you have downloaded the necessary `LitPop` and `EIOPA` datasets and placed them in the `.data/` directory as per the instructions in `loader.ipynb`.

> Note that for large countries with many NUTS3 regions, the spatial joins and graph loading may require significant memory in your Neo4j instance. Consider starting with a smaller country (e.g., Belgium) for initial testing.