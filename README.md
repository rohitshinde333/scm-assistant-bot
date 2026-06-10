# SCM Assistant Bot
> RAG + SQL powered Supply Chain chatbot built with Flowise | Trinamix Hiring Task TX-JrAI-003

---

## Quick Links

| | |
|---|---|
| **Public Chatbot URL** | `https://cloud.flowiseai.com/chatbot/daec4a2d-405a-4140-9df0-9efeac2d4a4c` |
| **GitHub Repository** | `https://github.com/rohitshinde333/scm-assistant` |

---

## Models Used

| Component | Model / Provider |
|---|---|
| **LLM** | Google Gemini `gemini-3.1-flash-lite-preview` |
| **Embeddings** | Google Gemini `gemini-embedding-001` |
| **Vector Store** | Pinecone (Dense, cosine, 3072 dimensions) |
| **Database** | Supabase PostgreSQL |

---

## Architecture

This chatbot uses a hybrid RAG (policy document) + SQL Agent (csv document) architecture built in Flowise Agentflow:
CSV file contains around 2000 records and examples queries asked have complex aggregation and filter requirements which satisfied by embedding CSV file. So I chose to use SQL for CSV retrieval and created custom tool. Took Agent node, added document store for policy document and CSV SQL tool. In order to aid complex arithmatic calucalations, I added calculator tool to agent. 
```
User Question
      ↓
  Agent (Gemini LLM)
  ┌────────────────┐
  │                │
  ▼                ▼
SQL Tool      Document Store
(Supabase)    (Pinecone RAG)
  │                │
  ▼                ▼
CSV Data      Policy PDF
Aggregations  Governance Rules
```

- SQL Tool — custom JavaScript tool that executes PostgreSQL queries against Supabase for data aggregation questions
- Document Store — Pinecone vector store with policy PDF chunks for governance rule retrieval
- Agent — dynamically decides which tool to use based on question type, retrieves policy first before querying data

---

## Data Files

| File | Description |
|---|---|
| `supplier_performance_data.csv` | 2,000 purchase orders · 116 suppliers · 27 columns |
| `SupplyChain_Governance_Policy_v3.2.pdf` | 10-section supplier governance policy |

---

## Chunk Configurations

Two configurations were tested to evaluate the effect of chunk size on answer quality.

Note: Chunking was used for the PDF policy document loaded into Pinecone. The CSV data was loaded into Supabase PostgreSQL for accurate SQL-based aggregation.

---

### Config 1 — Primary (Used in Production)

#### Embedding Settings
| Setting | Value |
|---|---|
| Provider | Google Gemini |
| Model | `gemini-embedding-001` |
| Task Type | `RETRIEVAL_DOCUMENT` (upsert) / `RETRIEVAL_QUERY` (query) |
| Strip New Lines | Off |

#### Pinecone Index Settings
| Setting | Value |
|---|---|
| Vector Type | Dense |
| Dimensions | 3072 |
| Metric | cosine |

#### PDF File
| Setting | Value |
|---|---|
| Chunk Size | 800 |
| Overlap | 100 |
| Separators | `["\n\n", "\n", ". ", "; ", ","]` |
| Chunk Count | 23 |
| Result | Full policy sub-sections intact per chunk, tables and SLA rules preserved |

---

### Config 2 — Experimental

#### PDF File
| Setting | Value |
|---|---|
| Chunk Size | 400 |
| Overlap | 80 |
| Separators | `["\n\n", "\n", ". ", "; ", ","]` |
| Chunk Count | 42 |
| Result | Policy sub-sections split further — individual rules retrievable but cross-section context lost |

---

### Observations

| Metric | Config 1 | Config 2 |
|---|---|---|
| PDF chunk count | 23 | 42 |
| Row integrity | Full rows intact | Some rows split |
| Policy section integrity | Sections intact | Sections fragmented |
| Q1 answer accuracy | Good | Degraded |
| Q2 answer accuracy | Good | Degraded |
| Q5 answer accuracy | Good | Acceptable |
| Overall verdict | Recommended | Not recommended |

Key finding: Config 1 (800/100) keeps full policy sub-sections intact per chunk — tables, SLA rules, and penalty structures stay together, making retrieval accurate for cross-referencing questions. Config 2 (400/80) increases chunk count from 23 to 42, fragmenting policy sub-sections across multiple chunks and causing the LLM to miss context when answering questions that span multiple rules within the same section.

---

## Sample Q&A (Verbatim Chatbot Answers)

Note on dataset versioning: The expected answers in the task brief were generated from a different version of the supplier dataset. All answers below are computed from the provided `supplier_performance_data.csv` and are correct for that dataset.

---

### Q1 — Tier-3 Suppliers with Active Disruption Flag
> Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

**Answer:**
```
According to the Supplier Governance Policy (§9), the response level for an active disruption is 
determined by the supplier's risk level:

Level 1 — Monitor: Low Risk suppliers.
Level 2 — Manage: Medium Risk suppliers.
Level 3 — Activate: High Risk suppliers (or any supplier with two simultaneous disruption flags).

Based on the analysis of Tier-3 suppliers with active disruption flags, the following response 
levels apply:

Supplier Name          | Supplier ID | Risk Level       | Applicable Response Level
Quetzal Textiles       | SUP-101     | High / Medium    | Level 3 / Level 2
Vistula Pack Sp        | SUP-075     | High / Medium    | Level 3 / Level 2
Deccan Polymer Mfg     | SUP-014     | High / Medium    | Level 3 / Level 2
Longhua Polymer Works  | SUP-007     | High / Medium    | Level 3 / Level 2
Visayas Textile Co     | SUP-042     | High / Medium    | Level 3 / Level 2
Lima Polymer SA        | SUP-096     | High / Medium    | Level 3 / Level 2
Buenos Aires Pack      | SUP-092     | High / Medium    | Level 3 / Level 2
Helios Pack Greece     | SUP-079     | High / Medium    | Level 3 / Level 2
Bogota Pack Ltda       | SUP-094     | High / Medium    | Level 3 / Level 2
Bohai Electronics      | SUP-010     | High / Medium    | Level 3 / Level 2
Deltaforge Vietnam     | SUP-018     | Medium           | Level 2
DaNang Metal Works     | SUP-022     | High             | Level 3
Patagonia Alloys SRL   | SUP-091     | High             | Level 3
Varna Electronics EAD  | SUP-078     | High / Medium    | Level 3 / Level 2
Archipelago PCB Corp   | SUP-041     | High             | Level 3
Yucatan Polymer Mfg    | SUP-089     | Medium           | Level 2
Orinoco Pack SAPI      | SUP-100     | Medium           | Level 2
Cerromax Mineria       | SUP-095     | High / Medium    | Level 3 / Level 2
Sibertek Molding       | SUP-045     | Medium           | Level 2
Maghreb Castworks      | SUP-080     | High / Medium    | Level 3 / Level 2
MeKong Pack Co         | SUP-020     | High             | Level 3
Dravex Components India| SUP-011     | High / Medium    | Level 3 / Level 2
Plataforma Metales SA  | SUP-090     | High / Medium    | Level 3 / Level 2
Sahyadri Alloy Tech    | SUP-017     | High / Medium    | Level 3 / Level 2

Note: As supplier risk levels can vary per purchase order, the response level should be applied 
based on the specific risk level associated with the active disruption flag for that supplier's 
current status.
```

---

### Q2 — Volume Rebate Program
> Which suppliers qualify for the annual Volume Rebate Program and how many are there?

**Answer:**
```
According to the Supplier Governance Policy (§4.2), the Volume Rebate Program is exclusively for 
Tier-1 suppliers who maintain an On-Time Delivery (OTD) rate of 93.0% or higher, a defect rate 
below 0.5%, and a sustainability score of 85 or higher for the full calendar year.

Based on the current supplier performance data, there are 25 suppliers that qualify for the annual 
Volume Rebate Program:

Supplier Name              | Supplier ID
Berliner Pack GmbH         | SUP-048
BlueMesa Polymers          | SUP-106
Borealis Composites        | SUP-062
Broken Hill Alloys         | SUP-044
Cascade Pack Corp          | SUP-107
Crestline Chemical Supply  | SUP-102
Fjord Polymer AS           | SUP-059
Formosa Circuit Co         | SUP-034
Hokkaido Alloy Tech        | SUP-028
Huabei Circuit Systems     | SUP-005
Kinki Precision Corp       | SUP-029
Kyushu Textile Co          | SUP-032
Maplewood Polymer Corp     | SUP-112
Munich Alloy Technik       | SUP-049
Nordloom Finland Oy        | SUP-060
Norrsken Textiles AB       | SUP-057
Orrentek Precision Mfg     | SUP-001
Osaka Polymer Systems      | SUP-030
Ostwind Composites AG      | SUP-064
PrecisionForge Taiyuan     | SUP-033
Rheinwerk Precision AG     | SUP-047
Ridgecroft Textiles        | SUP-104
Sendai Mech Works          | SUP-031
Solveig Eco Packaging      | SUP-056
Straits Packaging Hub      | SUP-037
```

---

### Q3 — Regional PO Concentration
> Which region has the highest total PO value, and does it breach the concentration limit?

**Answer:**
```
According to the Supplier Governance Policy (§5.3), no single region may account for more than 
45% of the total annual procurement spend.

Based on the current data:

Total Procurement Spend: USD 356,045,248.18

Regional Spend Breakdown:
- APAC: USD 131,620,356.14 (Highest)
- EMEA: USD 94,122,434.32
- LATAM: USD 83,971,664.94
- NA: USD 46,330,792.78

The APAC region has the highest total PO value. Calculating its concentration:
(131,620,356.14 / 356,045,248.18) * 100 = 36.97%

Conclusion: The APAC region accounts for 36.97% of the total spend, which is below the 45% 
concentration limit. Therefore, there is no breach of the concentration policy.
```

---

### Q4 — Supplier Watch List
> Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

**Answer:**
```
According to the Supplier Governance Policy (§3.4), any supplier with a Compliance Score below 60 
is placed on the Supplier Watch List (SWL). This status restricts new purchase order (PO) issuance 
to a maximum of 20% of the supplier's prior quarter volume.

There are 24 suppliers currently on the Supplier Watch List due to having a Compliance Score 
below 60:

Supplier Name              | Supplier ID
Orinoco Pack SAPI          | SUP-100
Quetzal Textiles           | SUP-101
Visayas Textile Co         | SUP-042
Longhua Polymer Works      | SUP-007
Vistula Pack Sp            | SUP-075
Deltaforge Vietnam         | SUP-018
DaNang Metal Works         | SUP-022
Varna Electronics EAD      | SUP-078
Bogota Pack Ltda           | SUP-094
Maghreb Castworks          | SUP-080
Helios Pack Greece         | SUP-079
Yucatan Polymer Mfg        | SUP-089
Buenos Aires Pack          | SUP-092
Archipelago PCB Corp       | SUP-041
Bohai Electronics          | SUP-010
Deccan Polymer Mfg         | SUP-014
Dravex Components India    | SUP-011
Plataforma Metales SA      | SUP-090
Lima Polymer SA            | SUP-096
Sibertek Molding           | SUP-045
Patagonia Alloys SRL       | SUP-091
Sahyadri Alloy Tech        | SUP-017
MeKong Pack Co             | SUP-020
Cerromax Mineria           | SUP-095

Note: The audit overdue criterion (§7.1) was excluded from this analysis as the dataset covers 
2023-2024 purchase orders. Applying overdue thresholds against the current date would incorrectly 
flag all suppliers as overdue. SWL status here is determined solely by Compliance Score < 60 
per §3.4.
```

---

### Q5 — Highest Average Defect Rate by Category
> Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

**Answer:**
```
According to the Supplier Governance Policy (§3.2 and §5.1), the maximum permissible defect rate 
for Tier-2 suppliers is 2.50%.

Based on the analysis of supplier performance data:

- Product Category with the highest average defect rate: Packaging Materials (1.91%)
- Comparison: The average defect rate of 1.91% is below the 2.50% Tier-2 limit.

Therefore, the product category with the highest average defect rate does not exceed the Tier-2 
limit.
```

---

## What I'd Improve

**1. Text-to-SQL instead of schema-guided prompting**
A proper production solution would use a Text-to-SQL framework (like LangChain SQL Agent or Vanna.ai) that automatically introspects the schema and generates accurate SQL for any question — not just ones created by LLM.

**2. Supplier master table**
The CSV has 2,000 PO-level rows with repeated supplier attributes. A normalized database with a separate suppliers table (one row per supplier) and a purchase_orders table would eliminate ambiguity in per-supplier metric calculations and make queries more reliable.

**3. Metadata filtering in Pinecone**
Key columns like Contract_Tier, Region, Risk_Level could be promoted to Pinecone metadata fields, enabling fast pre-filtering before semantic search — reducing noise in retrieved chunks.

**4. Hybrid search**
Combining dense vector search (semantic) with sparse BM25 keyword search in Pinecone would improve retrieval for exact supplier names and policy section references.

**7. Dataset versioning**
The provided CSV appears to be a different version from what the sample answers were generated from (different regional totals, supplier counts, risk levels). A production system would version datasets and regenerate ground-truth answers with each update.

---

## Security Notes

- Supabase anon key used (read-only public access)
- Pinecone API key stored in Flowise credentials vault