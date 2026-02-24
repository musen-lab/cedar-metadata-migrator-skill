---
name: cedar-metadata-harmonizer
description: >
  Transform legacy metadata records into compliant versions that adhere to a CEDAR metadata
  template specification. Use this skill whenever the user asks to transform, migrate, harmonize,
  clean up, or convert metadata records to match a CEDAR template. Also trigger when the user
  mentions CEDAR URLs (repo.metadatacenter.org), metadata compliance, ontology harmonization,
  metadata migration, or wants to fix/update metadata fields to match a specification.
  Handles JSON and CSV input, single and batch records, ontology lookups via BioPortal,
  value correction, and produces a processing log for human review.
---

# CEDAR Metadata Harmonizer

Transform legacy metadata records so they comply with a target CEDAR template specification.
The agent autonomously fetches the template, analyzes each record, resolves field values
(including ontology lookups, misplaced values, format corrections), and outputs a fully
compliant record plus a processing log.

## When to Read Reference Files

- **Before your first harmonization**: Read `references/workflow.md` for the full step-by-step procedure.
- **For ontology lookup patterns**: Read `references/ontology-resolution.md`.
- **For log file format**: Read `references/log-format.md`.

## Quick Overview

### Inputs

1. **Legacy metadata record(s)** — JSON object, JSON array, or CSV file.
2. **CEDAR template URL** — e.g. `https://repo.metadatacenter.org/templates/<id>`.

### Outputs

1. **Transformed record(s)** — Same format as input (JSON → JSON, CSV → CSV).
2. **Processing log** — Markdown file: `<YYYY-MM-DD>_<input-file-name>_processing-log.md`

### High-Level Workflow

```
1. Fetch CEDAR template  →  understand required fields, types, ontology constraints
2. Parse input            →  detect format (JSON/CSV), single vs batch
3. Per record:
   a. Map legacy fields to template fields (fuzzy matching on names/descriptions)
   b. For ontology-constrained fields → resolve values via BioPortal MCP tools
   c. Detect misplaced values (e.g. disease name in tissue field) → relocate
   d. Infer missing values from surrounding context when possible
   e. Apply format corrections (date patterns, SI units, string patterns)
   f. Log every decision, flag low-confidence ones for human review
4. Assemble output in original format
5. Write processing log
```

### Key Principles

- **Best-guess + flag**: Produce a value for every field where a reasonable inference 
  exists, and log the rationale for human review. The one exception: if an 
  ontology-constrained field's legacy value has no viable match in the permissible value
  list, leave it empty and flag it — a wrong ontology term is worse than none.
- **Ontology-first**: When a template field specifies an ontology or branch constraint,
  always attempt a BioPortal lookup. Use the standardized label and IRI from the lookup,
  not the raw legacy value.
- **Format fidelity**: Output format matches input format. JSON in → JSON out. CSV in → CSV out.
- **Non-destructive**: Legacy fields that don't map to any template field are recorded in the
  processing log under an Unmapped Fields section, so no data is silently dropped. The output
  file contains only template-compliant fields.

## Proceed

Read `references/workflow.md` now and follow the procedure.
