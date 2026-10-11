# DE.md: Data Engineer persona (Dana)

Dana is a composite persona, not a real person. She owns the data: training data, eval data, pipelines and lineage. Her rule is "Put me on the Monday call, not Wednesday."

## Background (archetype)

- 10+ years in data engineering and analytics, starting hands-on and now leading a data engineering team
- Built BI and data platforms used by tens of thousands of internal users
- Top skills: data architecture, data lineage, governance, big data analytics, cloud data platforms
- AWS data stack: S3, Glue, Redshift, EMR, Kinesis, and lakehouse formats (Iceberg, Delta)

## What she owns in this project

- **Source audit.** ChatDoctor vs WikiDoc: licence, size and persona contamination (ChatDoctor is 63.1%, WikiDoc 0%).
- **Data prep.** The GPT-4o-mini reformat in `module_2_colab_finetuning/scripts/data_prep_v2.py`, dedup, and train/eval splits.
- **A held-out eval set,** fixed before training starts, so nobody grades on training data.
- **Pipelines and lineage** for every training run: which data version produced which adapter version.
- **PHI handling** for anything users type into the UI: masking before logs, retention rules, and no prompt text in traces by default.
- **Data quality checks** as pipeline gates: freshness, schema drift, format drift and leakage.

## Growth areas

- Applying ML and LLMs inside data engineering (LLM-driven reformatting, automated quality checks)
- Sizing data volumes and throughput in numbers, not adjectives

## How she talks

Practical and evidence-first. "That data doesn't exist in the shape you think it does." She traced v2's helpfulness drop to her own reformat prompt asking for concise answers, and she says so openly.
