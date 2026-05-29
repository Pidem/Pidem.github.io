---
layout: post
title: "Data as a moat in HCLS"
subtitle: "A primer for fellow AI Specialists"
date: 2026-05-05
---
*A primer for fellow AI Specialists and other non-Data engineers*

---

## Three stories about private data

**Story 1: Identifying biomarkers** A mid-stage biotech runs a Phase II trial for a PD-1 inhibitor in non-small cell lung cancer. The trial fails — overall response rate is 12%, well below the 25% threshold for advancement. The program is about to be killed. But a translational science team runs multiplex immunofluorescence on archived tumor biopsies and discovers that patients with high spatial co-localization of CD8+ T-cells and PD-L1+ tumor cells responded at 47%. There's a candidate biomarker — but it's based on 83 patients from a single trial. Is it real, or a statistical fluke?

To validate, they need to check whether the same pattern holds in historical cohorts. The problem: they have H&E slides from three prior trials (2,400 patients total), but no multiplex staining — and re-staining archived tissue is expensive, slow, and destructive. So they run the H&E slides through a pathology foundation model, extract morphological embeddings, and train a classifier to predict the spatial immune phenotype from H&E alone. The model identifies a morphological signature that correlates with the mIF-defined biomarker at AUC 0.81. They validate the response association in the historical cohorts. The signal replicates.

The company redesigns Phase III around this subpopulation, enrolls 400 patients instead of 1,200, and gets an approval that generates $2B in peak-year revenue. Total time from failed trial to validated biomarker: 14 months — most of it spent not on science, but on data wrangling. The mIF data lived in one system, the clinical outcomes in another, the historical H&E slides in a third, and nobody had built the crosswalk between patient identifiers across trials.

**Story 2: Manufacturing Challenges** A biologics manufacturer loses a production batch of a monoclonal antibody. The batch passed every in-process control but failed final release testing for aggregation. Nobody knows why. The answer is buried in the intersection of three datasets: bioreactor sensor logs showing a subtle pH excursion at hour 37, a raw material lot that had slightly different glycosylation on the CHO cell feed, and an environmental record showing the facility HVAC cycled unexpectedly during a heat wave. Each dataset lives in a different system. The manufacturing team has access to the bioreactor data. The QC team has the raw material records. Facilities has the HVAC logs. Nobody has all three. The investigation takes 4 months. The next batch fails for the same reason.

**Story 3: The model that couldn't eat.** A computational biology team builds a foundation model for predicting drug-target interactions. The team has access to 2.3 million proprietary compound-assay pairs, a dataset that no one else has access to. But the assay data lives in a 15-year-old Oracle database managed by the chemistry informatics team. The compound structures are in a different system. The protein target annotations are in a third. Getting a training-ready dataset requires 6 weeks of back-and-forth, a data use agreement review, a manual export, and a custom ETL script that breaks every time the source schema changes. By the time the dataset is ready, the model architecture is already outdated. The team fine-tunes anyway, publishes a paper, and never retrains — because the cost of assembling the dataset again is too high.

Big Pharma companies experience these scenarios regularly. The proprietary data they take pride in is not accessible, not indexed, not governed. The moat is a swamp.

This essay is about how to drain the swamp.

## Why this matters financially

A Phase III oncology trial costs $1-2B and has a ~15% probability of success. Biomarker-driven patient enrichment can cut enrollment by 60%, push success probability above 50%, and shave 2-3 years off the timeline. Better patient stratification is worth hundreds of millions per program. But finding the biomarker requires linking molecular profiling, spatial biology, clinical outcomes, and historical trial data — four systems, four teams, four access policies. The companies that can run this analysis in days instead of months will find biomarkers their competitors miss.

The same pattern repeats across drug discovery (proprietary compound libraries + assay results + structural biology), manufacturing (sensor streams + raw material lots + environmental data), and real-world evidence (EHR notes + claims + registries). In each case, a foundation model trained on public data gives you a strong prior — general biological knowledge encoded in weights. But the posterior — what's specific to *your* targets, *your* patients, *your* process conditions — comes only from private data. The public model is the starting point. Your private data is the finishing move.

## Where the data lives today

A typical mid-to-large life sciences company's data landscape looks somewhat like this: 

| Domain | Systems | Typical owners | Access model |
|--------|---------|---------------|--------------|
| Genomics & omics | Internal pipelines → S3, BaseSpace, DNAnexus | Bioinformatics | Team-level bucket policies |
| Spatial biology | Vendor platforms (10x, Vizgen, NanoString) → shared drives or S3 | Research scientists | Whoever set it up |
| Clinical trial data | EDC (Medidata, Veeva CDMS) | Clinical operations | Formal data request process |
| Compound/assay data | ELN (Benchling, Dotmatics), LIMS, legacy Oracle DBs | Chemistry informatics | Application-level access |
| Structural biology | Internal databases, PDB submissions | Structural biology team | Ad hoc |
| Manufacturing | SCADA/PI historian, MES, LIMS, paper records | Manufacturing/QC | Air-gapped from R&D (GxP) |
| Medical imaging | PACS (radiology), vendor platforms (PathAI, Proscia) | Pathology/radiology | DICOM routing rules |
| Real-world evidence | Claims aggregators (Optum, Flatiron), EHR extracts | HEOR/RWE team | Licensed datasets with use restrictions |
| Clinical notes | EHR systems (Epic, Cerner) | Health system partners | BAAs and DUAs |
| Regulatory documents | Document management (Veeva Vault) | Regulatory affairs | Role-based within the system |
| Lab notebooks | ELN or scanned paper | Individual scientists | Often personal |
| Wearables/sensors | Device platforms, custom ingestion | Digital health team | Study-specific |

Count the systems. Count the teams. Count the access models. Now ask: *how do I run a query that spans three of these?*

You email someone, wait, get a CSV, email someone else, wait, get another CSV, join them in a Jupyter notebook, realize the patient IDs don't match because one system uses MRN and the other uses a study-specific identifier, email a third person to get the crosswalk table, wait, and eventually produce an analysis that took 2 months and is already stale.

This is not a technology problem. It's an organizational and architectural problem: each system was procured to solve one team's problem.

## Why foundation models make this urgent

For the past decade, the bottleneck in computational biology was *models* — we didn't have architectures that could learn from the scale and heterogeneity of biological data. That bottleneck is breaking.

The space has evolved rapidly in the last two years: 

- **Protein language models** (ESM-3, AlphaFold 3) that predict structure, function, and interactions from sequence
- **Genomic foundation models** (Evo, Nucleotide Transformer, HyenaDNA) that learn regulatory grammar from raw DNA
- **Pathology FMs** (Virchow, UNI, H-optimus-0, CONCH) that encode histology slides into rich embeddings
- **Single-cell FMs** (scGPT, Geneformer, scFoundation) that model cell state from transcriptomics
- **Molecular generation models** (diffusion models for 3D molecular design, language models for SMILES)
- **Clinical NLP models** (Med-PaLM, BioGPT, fine-tuned LLMs for clinical text extraction)

These models share a common property: **they are hungry for private data.**

A public pathology FM gives you general-purpose tissue embeddings. Fine-tuning on your proprietary trial cohort — where you have both the slides AND the treatment outcomes — gives you a predictive biomarker. The public model is the starting point. Your private data is the finishing move.

But feeding private data to these models requires:
1. **Discovery** — the ML team needs to know the data exists
2. **Access** — they need permission to use it, granted in hours not weeks
3. **Linkage** — the pathology slides need to be connected to the clinical outcomes, which requires a patient-level join across systems
4. **Governance** — the access needs to be auditable, revocable, and compliant with the consent under which the data was collected
5. **Repeatability** — when the next model arrives in 6 months, the same data needs to be accessible without re-doing the entire process

If any of these steps requires a manual process, the feedback loop stalls. And a stalled feedback loop means your private data isn't compounding — it's depreciating.

### The compounding feedback loop

[Alex Telford](https://atelfo.github.io/2024/09/17/a-new-breed-of-biotech.html) distinguishes two classes of biotech:

**Class I biotechs** are founded on a single biological insight — a target, a mechanism, a clever molecule. They're disposable rockets. Run the experiment, see if it works. These companies have little incentive to invest in durable data infrastructure because their most likely outcome is acquisition or failure.

**Class II biotechs** are built around computation and high-volume proprietary data generation. Companies like Recursion, Isomorphic Labs, Tempus, and the spatial biology startups. Their thesis is that each learning cycle generates data that feeds back into the platform and improves it for the future.

The Class II bet is that you can break the linear relationship between capital and output. Instead of "more money = more experiments = maybe one works," you get "more data = better models = more efficient experiments = compounding returns."

But the feedback loop only compounds if the infrastructure supports it:

1. **Trial outcomes** flow back to research → "why did it work in these patients?"
2. Research generates **hypotheses** tested against historical data → "do we see this pattern in prior cohorts?"
3. Hypotheses inform **next trial design** → "enrich for this biomarker"
4. The trial generates **new multimodal data** that enriches the platform
5. **New models** consume the enriched data → better predictions → better trial design → repeat

Each step crosses team boundaries, data modalities, access levels, and legal agreements. If any link requires a 6-week data request, the loop doesn't compound. It stalls.

## What "governance at scale" actually means for unstructured data

Most data governance conversations in life sciences assume the data is tabular.

Traditional governance tools were designed to govern *tables* — column-level security, row-level filtering, cell-level masking. This works for claims data, clinical trial metadata, and derived feature tables. But most of the valuable data in life sciences isn't a table:

- A 5 GB pathology slide
- A point cloud of 11 million spatial transcripts
- A 200 GB whole genome sequence
- A cryo-EM density map
- A bioreactor time-series with 50 sensor channels at 1-second resolution
- A 300-page regulatory submission PDF

You can't do column-level filtering on a FASTQ file. And yet, you still need to govern it. You need to know: who can discover it exists? Who can access the raw files? What consent agreement covers it? Can it be used for model training, or only for the specific study it was collected for?

The core architectural insight: **separate the governance of metadata from the governance of data.**

### Pattern 1: Inventory tables as the governance boundary

For every collection of unstructured data, maintain a structured catalog table:

```
| sample_id | patient_id | modality      | study_id  | classification | consent_scope   | s3_path |
|-----------|-----------|---------------|-----------|----------------|-----------------|---------|
| S001      | P1234     | H&E           | TRIAL-042 | PHI            | research_only   | s3://…  |
| S001      | P1234     | spatial_tx    | TRIAL-042 | PHI            | research+model  | s3://…  |
| S002      | P5678     | WES           | TRIAL-042 | de-identified  | unrestricted    | s3://…  |
| S003      | P9012     | cryo-EM       | DISC-007  | non-human      | unrestricted    | s3://…  |
```

Govern this table with fine-grained access control (row-level, column-level). The ML team sees rows where `consent_scope IN ('unrestricted', 'research+model')`. A partner sees only rows matching their data use agreement. The actual S3 files are accessed through **credential vending** — short-lived credentials scoped to exactly the S3 prefixes that the user's governance permissions allow.

The user never gets broad S3 access. They get a temporary key that works for exactly the files their permissions allow. When permissions change, access changes instantly — no bucket policies to update, no IAM roles to modify.

### Pattern 2: Extract → Structure → Govern

For data with extractable structure — clinical notes, batch records, regulatory documents, lab notebooks:

1. Raw document lands in object storage (governed by coarse-grained policy)
2. Extraction pipeline (OCR, NLP, or LLM-based) produces structured entities: drug names, adverse events, patient demographics, process parameters, regulatory findings
3. Structured output lands in a governed table with full fine-grained access control
4. Downstream consumers query the structured table, never touching the raw document

The raw data is the source of truth. The structured extraction is the governed, queryable interface. When better extraction models arrive, re-run the pipeline — the governance model doesn't change.

### Pattern 3: Embeddings as a governed query layer

When you embed unstructured data (pathology patches, document chunks, compound structures, genomic sequences) into vector representations, those embeddings live in a governed table:

```
| chunk_id | source_id | study_id | classification | embedding_vector | model_version |
```

Row-filter this table by `classification`, `study_id`, or any governance dimension. When a RAG system or similarity search queries the embeddings, it only retrieves vectors the requesting user is authorized to see. **Retrieval becomes permission-aware.**

This means a new foundation model can query across your entire corpus — but it only "sees" embeddings derived from data the requesting team has access to. No raw PHI is exposed. The governance boundary is enforced at the embedding layer.

### Pattern 4: The "new model" problem

When a new pathology FM arrives and needs 50,000 H&E slides for fine-tuning:

1. The training job assumes a role tagged with governance attributes: `modality=H&E`, `purpose=model_training`, `classification=de-identified`
2. The governance engine evaluates the inventory table: which rows match?
3. The job receives temporary credentials scoped to exactly those S3 prefixes
4. Training proceeds. No human approval needed per-file.

When the *next* model arrives six months later needing spatial transcriptomics data, you add a new tag value. The governance framework doesn't change — only the tag assignment. **New data types and new consumers plug into the same framework without re-architecture.**

## Making the feedback loop operational

The governance patterns above solve the *access* problem. But a compounding data asset also requires that teams can *discover* what exists and *build on* what others have produced. This is the producer/consumer model:

1. **Genomics team** processes raw sequencing data, publishes governed datasets to a shared catalog
2. **Clinical team** publishes trial outcome data with appropriate access controls
3. **ML team** discovers both datasets in the catalog, subscribes, gets access, trains a biomarker model
4. **ML team** publishes model predictions back to the catalog as a new governed dataset
5. **Clinical team** subscribes to predictions, uses them to inform next trial design
6. **Manufacturing team** publishes process data, ML team correlates with batch outcomes
7. **Regulatory team** queries lineage: "which datasets and models contributed to this submission?"

Each step is governed, audited, and traceable. The catalog is the connective tissue. Without it, you have teams producing valuable data that other teams don't know exists.

The technical substrate for this is a **lakehouse architecture**: open table formats (Apache Iceberg) on object storage, with a unified catalog that any compute engine can query — whether that's SQL analytics, Spark pipelines, or model training jobs. The key properties:

- **Engine-agnostic**: Your bioinformatics team uses Spark. Your clinical analysts use SQL. Your partner uses Snowflake. All read the same governed tables.
- **Time travel**: Every query is reproducible. Regulatory submissions can reference the exact dataset version used.
- **Schema evolution**: When a new assay produces a new column, existing queries don't break.
- **Zero-copy sharing**: Partners see a pointer to your data in their own environment. No exports, no CSVs, no drift.


## Conclusion

The winners in HCLS AI won't have the best models. Models are commoditizing — the same architectures are available to everyone, and foundation models are improving on a 6-month cadence regardless of what you do.

The winners will be the ones who can answer a question like: *"Show me all patients across our last three trials who had high CD8+ T-cell infiltration in the tumor microenvironment, carried a specific KRAS mutation, and progressed within 6 months — then fine-tune a response prediction model on that cohort by Friday."*

That question spans genomics, spatial biology, clinical outcomes, and ML — four teams, four systems, three consent agreements. If your answer is "we'll need to set up a meeting to discuss the data request process," you've already lost.

Private data is the moat: Infrastructure is what makes it liquid. And liquidity is what makes it compound.

---

## Sources & further reading

- [Cancer has a surprising amount of detail](https://www.owlposting.com/p/cancer-has-a-surprising-amount-of) — Abhishaike Mahajan, Owl Posting (Oct 2025)
- [Data-driven biological research: A biotech's guide to establishing a strong data strategy](https://www.benchling.com/blog/biotech-guide-to-data-driven-rd) — Benchling (Aug 2023)
- [A new breed of biotech](https://atelfo.github.io/2024/09/17/a-new-breed-of-biotech.html) — Alex Telford (Sep 2024)
- Multimodal World Models for Drug Discovery — Eshed Margalit, Noetik.ai (Stanford CS 25 lecture, 2025)

*Last updated May 2026*
