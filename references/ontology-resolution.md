# Ontology Resolution Reference

How to resolve legacy free-text values into standardized ontology terms using the
BioPortal MCP tools.

## Available Tools

### `term_search_from_branch`

Search within a specific branch of an ontology. Use when the template field has
`permissible_values` of type `"branch"`.

```
term_search_from_branch(
  search_string="breast cancer",
  ontology_acronym="DOID",
  branch_iri="http://purl.obolibrary.org/obo/DOID_4"
)
```

Parameters:
- `search_string`: The legacy value to resolve (clean it first — see below).
- `ontology_acronym`: From the template's `permissible_values[].ontology_acronym`.
- `branch_iri`: From the template's `permissible_values[].branch_iri`.

### `term_search_from_ontology`

Search across an entire ontology. Use when the template field has `permissible_values`
of type `"ontology"`.

```
term_search_from_ontology(
  search_string="breast cancer",
  ontology_acronym="DOID"
)
```

Parameters:
- `search_string`: The legacy value to resolve.
- `ontology_acronym`: From the template's `permissible_values[].ontology_acronyms[]`.
  If multiple acronyms are listed, try each one and pick the best overall match.

## Value Cleaning Before Lookup

Before sending a value to BioPortal, clean it:

1. **Trim whitespace** and normalize casing.
2. **Strip qualifiers**: `"primary breast cancer"` → try both `"primary breast cancer"` and `"breast cancer"`.
3. **Expand abbreviations**: `"HCC"` → `"hepatocellular carcinoma"`, `"AML"` → `"acute myeloid leukemia"`.
   Use domain knowledge to expand common biomedical abbreviations.
4. **Remove noise**: trailing punctuation, parenthetical annotations, extra whitespace.

## Selecting the Best Match

BioPortal returns a list of candidate terms. Pick the best one using this priority:

1. **Exact label match** (case-insensitive): the result's `prefLabel` equals the search string.
2. **Synonym match**: the search string appears in the result's synonym list.
3. **Partial match with highest relevance**: BioPortal ranks results — prefer higher-ranked ones.
4. **Broader/narrower term**: If no exact match, consider whether a broader or narrower
   term is acceptable. Prefer the narrower (more specific) term when both fit.

From the selected match, extract:
- **label**: The `prefLabel` — this is the standardized name.
- **IRI**: The term's `@id` or `links.self` — this is the canonical identifier.

## Handling Multiple Ontologies

When `ontology_acronyms` lists multiple ontologies (e.g., `["DOID", "NCIT"]`):

1. Search each ontology.
2. Prefer the ontology that the template lists first.
3. If the first ontology has no match but the second does, use the second.
4. Log which ontology was used.

## When Lookup Fails

If no reasonable match is found after trying all available ontologies:

1. Enter `null`.
2. In the log, flag it as `NO_ONTOLOGY_MATCH` with:
   - The original value
   - Which ontologies were searched
   - The closest candidates (if any) so the human reviewer can decide

## Common Pitfalls

- **Overly specific searches**: `"invasive ductal carcinoma of the left breast stage IIB"`
  won't match. Try progressively shorter versions: `"invasive ductal carcinoma"` → `"ductal carcinoma"`.
- **Wrong ontology for the value**: A disease name searched in a tissue ontology will fail.
  This is actually a signal that the value is misplaced — see the misplaced value detection
  in `references/workflow.md`.
- **Deprecated terms**: BioPortal may return obsolete terms. Prefer non-obsolete results.
