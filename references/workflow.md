# Workflow Reference

Complete step-by-step procedure for transforming legacy metadata records.

## Step 1: Fetch and Analyze the CEDAR Template

Use the `get_cedar_template` MCP tool with the user-provided CEDAR URL.

```
get_cedar_template(template_id="<full CEDAR URL or ID>")
```

## Step 2: Parse the Input

Detect the input format and structure:

| Input | Detection | Mode |
|-------|-----------|------|
| JSON object `{ ... }` | Top-level is object | Single record |
| JSON array `[ { ... }, ... ]` | Top-level is array | Batch |
| CSV file (`.csv`) | File extension or comma-delimited content | Batch (each row = 1 record) |

For CSV: parse headers as field names, each row as a record. Use Python's `csv` module or
`pandas` for robust parsing.

## Step 3: Field Mapping

For each legacy record, map its keys to template field names. Use this priority order:

1. **Exact match** — legacy key equals template `name` (case-insensitive).
2. **Label match** — legacy key equals template `label` (case-insensitive).
3. **Synonym/fuzzy match** — legacy key is a common synonym or abbreviation of the
   template field. Examples: `"sex"` ↔ `"gender"`, `"cell_line"` ↔ `"cellLine"`, `"dob"` ↔ `"age"`.
4. **Description match** — the legacy value semantically fits a template field's description
   better than its current mapping. This is how you catch misplaced values.

### Detecting Misplaced Values

After initial mapping, cross-check each value against its target field's description and
ontology constraint. If a value clearly belongs to a different field:

- Example: legacy field `"tissue"` contains `"breast cancer"` — this is a disease, not a tissue.
- Action: move `"breast cancer"` to the `disease` field, and attempt to infer the actual
  tissue from context (e.g., if disease is breast cancer, tissue might be `"breast"`).
- Log this relocation with rationale.

## Step 4: Value Resolution (Per Field)

For each mapped field, resolve the value according to the template's constraints.
See `references/ontology-resolution.md` for detailed ontology lookup procedures.

### 4a. Ontology-Constrained Fields

If the field has `permissible_values`:

1. Take the legacy value as a search string.
2. Call the appropriate BioPortal tool (`term_search_from_branch` or `term_search_from_ontology`).
3. From the results, pick the best match — prefer exact label matches, then partial matches.
4. Output the **standardized label only** (plain string). Do not include the ontology IRI
   in the output record — IRIs are recorded in the processing log for traceability.
5. If no good match is found, set the value to `null` (not `""`) and flag it in the log.

### 4b. Datatype Enforcement

Every value must conform to the field's datatype as declared in the template schema.
This is critical — a value that looks correct but violates the datatype is invalid.

Rules:
- **String fields**: Output as a string. No special handling beyond ontology resolution
  and format corrections.
- **Numeric fields** (`number`, `integer`, `decimal`): Output must be a bare number with
  no embedded units. If the legacy value mixes a number with a unit (e.g., `"64 yr"`,
  `"5.2 kg"`), extract the numeric part and place it in the field. If the template has a
  companion unit field (e.g., `age_unit` alongside `age`), place the unit there. If no
  companion unit field exists, log the stripped unit so the reviewer is aware.
- **Boolean fields**: Normalize to `true` / `false`.
- **Date fields**: Normalize to the pattern specified in the template (e.g., ISO 8601).

Example:
- Legacy: `"age": "64 yr"`
- Template has: `age` (type: number) and `age_unit` (type: string)
- Output: `"age": 64`, `"age_unit": "year"`

### 4c. Free-Text String Fields

No ontology lookup and no numeric extraction needed. Apply format corrections only if
the template description implies a specific pattern:

- **Identifiers**: Preserve as-is unless clearly malformed.
- **Provider/address**: Preserve as-is.

### 4c. Missing Values

When a field exists in the template but has no corresponding legacy value:

1. Check if the value can be **inferred from context** in other fields.
   - Example: if `sex` is missing but `isolate` contains `"female_donor_42"`, infer `"Female"`.
   - Example: if `tissue` is missing but `disease` is `"hepatocellular carcinoma"`, infer `"liver"`.
2. If inferable, fill it in and flag as `INFERRED` in the log.
3. If not inferable and the field is required, set to `null` and flag as `MISSING_REQUIRED` in the log.
4. If not inferable and the field is optional, set to `null` and note as `MISSING_OPTIONAL` in the log.

## Step 5: Assemble Output

### JSON Output

For single records, output a JSON object. For batch, output a JSON array.

Each record's structure follows the template field names as keys. All values are plain —
ontology-constrained fields use the standardized label as a simple string, not a nested
object. Ontology IRIs are recorded only in the processing log for traceability.

Values must conform to the field's declared datatype: strings as strings, numbers as
bare numbers (no embedded units), booleans as `true`/`false`. Unresolved or missing
values must be `null`, never empty strings `""`.

```json
{
  "disease": "Breast Cancer",
  "tissue": "Breast",
  "age": 63,
  "age_unit": "year",
  "sex": "Female"
}
```

The output contains only template-compliant fields. Any legacy fields that could not be
mapped to the template are recorded in the processing log (see `references/log-format.md`),
not in the output file.

### CSV Output

Use the template field names as column headers. Ontology-constrained values use just the
standardized label. Only template fields appear in the output; unmapped legacy columns
are logged separately.

## Step 6: Write the Processing Log

Follow the format in `references/log-format.md`. The log captures every transformation
decision, flags concerns, and provides a human reviewer with a clear action list.

## Error Handling

- **Template fetch fails**: Inform the user, do not proceed.
- **BioPortal lookup returns no results**: Keep the original value, flag in log as `NO_ONTOLOGY_MATCH`.
- **Ambiguous mapping**: Make best guess, flag as `AMBIGUOUS_MAPPING` in log.
- **Malformed input**: Report which records/rows failed to parse, continue with valid ones.