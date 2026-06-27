# Reference: Expected Database Headers

This document describes the expected column names from each external database used in the pipeline. **Always run a header check before executing join or filter operations** — web servers update their output formats without notice.

Quick R check:
```r
colnames(read.csv("your_file.csv"))
# or for TSV:
colnames(read_tsv("your_file.tsv"))
```

---

## SwissADME

**URL:** https://www.swissadme.ch/  
**Input format:** Plain text, one per line: `SMILES IDENTIFIER` (space-separated, no header)  
**Output:** CSV

| Column used | Typical name | Notes |
|-------------|-------------|-------|
| Identifier | `Molecule` | The identifier you submitted (use PubChemCID) |
| Lipinski violations | `Lipinski..violations` | Integer 0–4 |
| GI absorption | `GI.absorption` | "High" or "Low" |
| BBB permeant | `BBB.permeant` | "Yes" or "No" — optional filter |
| LogP | `Consensus.Log.Po.w` | Filter: ≤ 5 |
| MW | `MW` | Filter: ≤ 500 Da |

> **Filter recommendations:**  
> Natural product extracts: `Lipinski..violations <= 1` and `GI.absorption %in% c("Yes", "High")`  
> Strict drug-likeness: `Lipinski..violations == 0` and `GI.absorption == "High"`

---

## ADMETlab 2.0

**URL:** https://admetmesh.scbdd.com/  
**Input format:** SMILES (paste or upload)  
**Output:** CSV or Excel

| Column used | Typical name | Notes |
|-------------|-------------|-------|
| SMILES | `smiles` | Input SMILES (used for joining back) |
| Mutagenicity | `Ames` | Probability 0–1; safe: < 0.5 |
| Liver injury | `DILI` | Probability 0–1; safe: < 0.5 |
| Cardiotoxicity | `hERG` | Probability 0–1; safe: < 0.5 |
| Acute toxicity | `h_HT` | Human hepatotoxicity; optional, safe: < 0.5 |

## ADMETlab 3.0

**URL:** https://admetlab3.scbdd.com/  
**Input format:** SMILES

| Column used | Typical name v3 | Notes |
|-------------|----------------|-------|
| SMILES | `SMILES` | Capitalized in v3 |
| Mutagenicity | `Ames_mutagenicity` | |
| Liver injury | `DILI` | |
| Cardiotoxicity | `hERG_blockers` | |

---

## SwissTargetPrediction (STP)

**URL:** https://www.swisstargetprediction.ch/  
**Input format:** SMILES (one per query)  
**Output:** CSV per compound  
**File naming convention used here:** `Targets<PubChemCID>.csv`

| Column used | Typical name | Notes |
|-------------|-------------|-------|
| Target protein | `Target` | Protein name |
| Gene symbol | `Gene Symbol` | HGNC gene symbol |
| UniProt ID | `Uniprot ID` | Used for cross-referencing GeneCards |
| Probability | `Probability*` | Already 0–1 decimal; filter: > 0 |
| Model accuracy | `Model accuracy` | 0–1 decimal |
| Common name | `Common name` | |

> **Note:** In older STP versions, probability was expressed as a percentage (e.g., 83.7%).
> In that case, divide by 100: `Probability = as.numeric(str_remove(Probability, "%")) / 100`

---

## SuperPred

**URL:** https://prediction.charite.de/  
**Input format:** SMILES or compound name  
**Output:** CSV per compound  
**File naming convention used here:** `Targets<PubChemCID>.csv`

| Column used | Typical name | Notes |
|-------------|-------------|-------|
| Target protein | `Target_name` | |
| UniProt ID | `uniprot_id` | |
| Probability | `probability` | May be 0–100 (percentage) or 0–1 — check! |
| Model accuracy | `model_acc` | |

> **Important:** Check whether `probability` values are > 1 (percentage scale) or ≤ 1 (decimal scale).
> If > 1, convert: `Probability = probability / 100`

---

## GeneCards

**URL:** https://www.genecards.org/  
**Input format:** Search term (e.g., "Hypertension", "Inflammation")  
**Output:** CSV (download button on results page)  
**File naming convention used here:** `GeneCards<Term>.csv`

| Column used | Typical name | Notes |
|-------------|-------------|-------|
| Gene symbol | `Gene Symbol` | HGNC symbol |
| UniProt ID | `Uniprot ID` | Used to join with target predictions |
| Relevance score | `Relevance score` | GeneCards score; recommended filter: > 5 |
| Description | `Gene Description` | |

> **Filter recommendation:** `Relevance score > 5` removes low-confidence associations.
> Increase to > 10 or > 20 for a stricter disease-specific analysis.

---

## STRING Database (via STRINGdb R package)

**Package:** `STRINGdb` (Bioconductor)  
**Input:** Gene symbol list as plain data.frame  
**Output:** Interaction data frame

| Column | Meaning |
|--------|---------|
| `from` | STRING internal ID (source protein) |
| `to` | STRING internal ID (target protein) |
| `combined_score` | Combined evidence score (0–1000) |

> **Score thresholds:**
> - 400 = medium confidence
> - 700 = high confidence (recommended)
> - 900 = very high confidence

After calling `get_interactions()`, translate IDs back to gene symbols using `left_join()` with the mapped table from `string_db$map()`.

---

## PubChem (via webchem R package)

Used in Step 01 to retrieve SMILES and CIDs from compound names.

| Function | Purpose |
|----------|---------|
| `get_cid(name, from = "name", match = "first")` | Get PubChem CID from compound name |
| `pc_prop(cid, properties = "SMILES")` | Get canonical SMILES from CID |

> **Why name lookup instead of CAS?**  
> A compound can have multiple CAS numbers registered across different databases (e.g., the racemate CAS vs. the pure enantiomer CAS). This causes silent mismatches when joining tables. PubChem name lookup resolves to a single canonical CID, which is then used as the primary identifier throughout this pipeline.
