# Network Pharmacology Cookbook

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Language: R](https://img.shields.io/badge/Language-R-276DC3.svg)](https://www.r-project.org/)

**Author:** Edgard Uriel López Chablé  
**Group:** Laboratorio de Investigación Química y Farmacológica de Productos Naturales, Posgrado en Ciencias Químico Biológicas, UAQ

A reproducible, step-by-step R pipeline for **network pharmacology analysis of plant extracts**, from raw compound data to pre-docking candidate prioritization. Developed and validated using phenolic compounds from *Sphaeralcea angustifolia*.

> **Why this cookbook?** Network pharmacology papers are common but reproducible, documented pipelines are rare — most groups publish results without code. This repository is designed so that a researcher can follow the workflow from zero, understand each methodological decision, and adapt it to their own extract and disease of interest.

---

## Pipeline Overview

```
Raw compound data (GC-MS matrix or curated compound list)
        ↓
[00] Setup         — R packages + project folder scaffold
        ↓
[01] Cleaning      — SMILES retrieval via PubChem, binarization by condition
        ↓
[02] ADME          — SwissADME / Lipinski rule-of-five filter
        ↓
[03] Toxicity      — ADMETlab2/3 (Ames, DILI, hERG)
        ↓
[04] Targets       — SuperPred / SwissTargetPrediction → consolidate by PubChem CID
        ↓
[05] Networks      — GeneCards intersection · Venn · Networks B/C/D/E
                     Topology metrics · Multi-centrality hub ranking
                     GO / KEGG / Reactome enrichment · Pathview maps
        ↓
[06] Pre-Docking   — Heatmap · UpSet · Hub bar chart
                     DGIdb drug–gene cross-validation
                     Docking candidate prioritization table

              [ FUTURE: Molecular Docking → Molecular Dynamics ]
```

---

## Folder Structure

```
network-pharmacology-cookbook/
├── LICENSE
├── README.md
├── .gitignore
├── 00_setup/               R packages + project scaffold
├── 01_data_cleaning/       SMILES retrieval, replicate collapsing, binarization
├── 02_adme_filtering/      SwissADME / Lipinski filtering
├── 03_toxicity_filtering/  ADMETlab2/3 toxicity screen
├── 04_target_prediction/   File naming convention + prediction consolidation
├── 05_network_analysis/    Disease target intersection, networks, enrichment
│   └── output/             Generated figures, tables, and pathview maps
├── 06_predocking_analysis/ Heatmaps, UpSet, centrality, DGIdb, candidate table
│   └── output/             Generated figures and candidate CSVs
├── example_data/           Ready-to-run example dataset (8 compounds, Sphaeralcea angustifolia)
│   ├── 04_targets/         Targets<PubChemCID>.csv files
│   └── 05_genecards/       GeneCards disease target CSVs
└── reference/              Database headers, expected column formats
```

---

## Quick Start (Example Data)

Each `.Rmd` file has this flag at the top:

```r
USE_EXAMPLE_DATA <- TRUE   # ← flip to FALSE for your own project
```

When `TRUE`, scripts load the bundled 8-compound dataset (*Sphaeralcea angustifolia* phenolics). Set to `FALSE` and update file paths for your own data.

Run files **in order**: `00` → `01` → `02` → `03` → `04` → `05` → `06`.

> **Important:** All `.Rmd` files use `knitr::opts_knit$set(root.dir = normalizePath(".."))` in the setup chunk so that all paths are relative to the cookbook root — not the subfolder the file lives in. This is intentional.

---

## Input Data Scenarios

**Scenario A — GC-MS matrix:**  
Compound × sample abundance matrix with replicate columns per condition (format: `R<n>-<CONDITION>`, e.g. `R1-LEA-ET`, `R2-FLO-AQ`). Run Steps 01–06.

**Scenario B — Curated compound list:**  
Table with compound names, PubChemCIDs, and SMILES (from literature or manual curation). Start at Step 02.

---

## Key Convention: `Targets<PubChemCID>.csv`

Target prediction files (one per compound) are named using the **PubChem Compound ID (CID)**, not the CAS number.

> **Why not CAS?** A single compound can have multiple valid CAS numbers registered across databases (racemate vs. enantiomer, anhydrous vs. hydrate, etc.). The PubChem CID is unique, stable, and unambiguous.

Example: Apigenin → `Targets5280443.csv`  
Find any CID: https://pubchem.ncbi.nlm.nih.gov/

---

## Supported Target Prediction Servers

| Server | URL | Notes |
|--------|-----|-------|
| SwissTargetPrediction | https://www.swisstargetprediction.ch/ | Select *Homo sapiens* |
| SuperPred | https://prediction.charite.de/ | SMILES or name input |

Both are free and accept SMILES input. Column names differ between servers and between versions — see `reference/database_headers.md` for the header reference and how to adapt the rename blocks.

---

## What Step 05 Produces

| Output | Description |
|--------|-------------|
| Venn diagram | Compound targets ∩ disease targets (GeneCards) |
| Network B | Bipartite compound–target graph |
| Network C | STRING PPI (REST API, no large file download) |
| Network Topology | Nodes, edges, density, clustering coefficient, avg path length |
| Network D | Hub gene subgraph — node size = degree, color = betweenness |
| HubGene_Table.csv | Degree + betweenness + closeness + composite score |
| GO / KEGG / Reactome | Enrichment tables + dotplots |
| Pathview maps | KEGG pathway PNG maps compiled to PDF |
| Network E | Tripartite cnetplot (Reactome) |

---

## What Step 06 Produces

| Output | Description |
|--------|-------------|
| Hub target bar chart | Targets ranked by number of compound interactions |
| UpSet plot | Shared target intersections between compounds |
| Heatmap | Compound × target binding probability matrix |
| DrugGene_Validation.csv | Known drugs per hub gene (DGIdb — aggregates DrugBank, ChEMBL, TTD) |
| **Docking_Candidates_Ranked.csv** | **Final ranked table for molecular docking** |
| GOChord (optional) | Gene–pathway chord diagram (GOplot) |

---

## Dependencies

See `00_setup/00_setup.Rmd` for the complete installation block.

**CRAN:** `tidyverse`, `igraph`, `ggraph`, `ggVennDiagram`, `pheatmap`, `UpSetR`, `RColorBrewer`, `httr`, `jsonlite`, `fs`, `magick`

**Bioconductor:** `clusterProfiler`, `org.Hs.eg.db`, `ReactomePA`, `enrichplot`, `pathview`

**Optional:** `GOplot` (GOChord diagram)

---

## License

MIT © 2026 Edgard Uriel López Chablé — see [LICENSE](LICENSE).  
Code developed entirely by Edgard Uriel López Chablé. Free to use and adapt with attribution.
