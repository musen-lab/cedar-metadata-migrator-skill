# Processing Log Format Reference

The processing log is a Markdown file that documents every transformation decision,
flags concerns, and gives a human reviewer a clear action list.

## File Naming

```
<YYYY-MM-DD>_<input-file-name>_processing-log.md
```

- Date: today's date in `YYYY-MM-DD` format.
- Input file name: the original file name, converted to kebab-case, without extension.
- All segments joined with underscores, internal segments in kebab-case.

Examples:
- Input: `samples.json` on 2026-02-18 → `2026-02-18_samples_processing-log.md`
- Input: `BioSample_Data.csv` on 2026-02-18 → `2026-02-18_bio-sample-data_processing-log.md`

## Log Structure

```markdown
# Metadata Transformation Log

- **Date**: YYYY-MM-DD
- **Input file**: `<original filename>`
- **Input format**: JSON / CSV
- **Records processed**: N
- **Template**: `<CEDAR template name>` ([link](<CEDAR URL>))

## Summary

| Metric | Count |
|--------|-------|
| Total records | N |
| Fully resolved (no flags) | N |
| Records with warnings | N |
| Fields resolved via ontology | N |
| Fields inferred from context | N |
| Misplaced values relocated | N |
| Unresolved fields | N |
| Unmapped legacy fields | N |

## Action Required

Items below need human review. Each item includes the record identifier,
the field, the agent's best guess, and the concern.

### ⚠️ HIGH — Likely incorrect or ambiguous

- **Record N, field `<name>`**: <description of concern>
  - Legacy value: `<original>`
  - Assigned value: `<what the agent chose>`
  - Reason for concern: <why this needs review>

### 🔶 MEDIUM — Reasonable guess, but uncertain

- **Record N, field `<name>`**: <description>
  - Legacy value: `<original>`
  - Assigned value: `<chosen>`
  - Reason: <explanation>

### 🔵 LOW — Informational

- **Record N, field `<name>`**: <description>
  - Note: <what happened, e.g. "value inferred from disease field">

## Per-Record Details

### Record 1

| Field | Legacy Value | Resolved Value | IRI | Resolution Method | Flag |
|-------|-------------|----------------|-----|-------------------|------|
| disease | breast cancer | Breast Cancer | DOID:1612 | Ontology lookup (DOID) | ✅ |
| tissue | — | Breast | NCIT:C12971 | Inferred from disease | 🔶 INFERRED |
| sex | F | Female | PATO:0000383 | Ontology lookup (BAO→PATO) | ✅ |
| age | 63 years | 63 | — | Datatype enforcement (extracted number) | ✅ |
| age_unit | — | year | — | Extracted from legacy `age` value | 🔵 FORMAT_CORRECTED |

### Record 2
...

## Unmapped Fields

Legacy fields that have no corresponding template field. These are excluded from
the output file and preserved here so no data is silently lost. The reviewer should
decide whether to discard them or incorporate them elsewhere.

| Record | Legacy Field | Value | Notes |
|--------|-------------|-------|-------|
| 1 | custom_legacy_field | some value | No template field match found |
| 1 | internal_batch_id | BTX-2024-0042 | Likely an internal tracking ID |
| 2 | lab_notes | cultured 3 days | May be relevant to biomaterial_provider |
```

## Flag Types

| Flag | Meaning | Action Needed |
|------|---------|---------------|
| ✅ | Resolved confidently | None |
| 🔶 INFERRED | Value guessed from context | Verify guess is correct |
| ⚠️ AMBIGUOUS_MAPPING | Multiple possible field mappings | Confirm correct field |
| ⚠️ NO_ONTOLOGY_MATCH | BioPortal returned no results | Manually assign ontology term |
| ⚠️ RELOCATED | Value moved from one field to another | Confirm relocation is correct |
| 🔵 MISSING_OPTIONAL | Template field had no source data | Optionally fill in |
| ⚠️ MISSING_REQUIRED | Required template field has no source data | Must fill in |
| 🔵 FORMAT_CORRECTED | Value reformatted (e.g., date, units) | Verify formatting |
| 🔵 DATATYPE_CORRECTED | Unit stripped from numeric field or type coerced | Verify value and unit placement |

Unmapped legacy fields are not flagged inline — they appear in their own
**Unmapped Fields** section at the end of the log.

## Guidelines for Writing Log Entries

- Be specific: say exactly what happened, not just that something was flagged.
- Include both the original and resolved values so the reviewer can compare.
- For ontology lookups, include the term ID (e.g., DOID:1612) alongside the label.
- Group high-priority items at the top so reviewers see critical issues first.
- Keep per-record tables concise — one row per field, not lengthy prose.