# CEDAR Metadata Migrator

A Claude Code custom skill that transforms legacy metadata records into compliant versions that adhere to [CEDAR](https://metadatacenter.org/) template specifications.

Given a metadata record object (JSON or CSV) and a CEDAR template ID in the prompt, the skill will:

- **Map legacy fields** to template fields using exact, label, fuzzy, and semantic matching
- **Resolve ontology terms** via BioPortal lookups for fields with ontology constraints
- **Detect misplaced values** and relocate them to the correct fields
- **Enforce data types** including strings, numbers, booleans, and dates
- **Generate a processing log** documenting every transformation decision with flags for human review

The output is always in the same format as the input (JSON in, JSON out; CSV in, CSV out), and every decision is logged transparently so nothing is silently lost.

## Requirement

Install the [cedar-mcp](https://github.com/musen-lab/cedar-mcp) tool into Claude Desktop beforehand.

## Installing in Claude Desktop

### Prerequisites

- [Claude Desktop](https://claude.ai/download) version 1.1.x or later

### Step-by-step

1. Download `cedar-metadata-migrator.zip` from the latest [GitHub Release](../../releases/latest).

2. Go to "Customize" > "Skills" in the Claude Desktop app

3. Look for the "+" sign and select "Upload a skill"

4. Drop or upload the ZIP file

5. Restart Claude Desktop to pick up the new skill.

6. Verify the skill is loaded by asking Claude:
   ```
   Migrate my metadata records below to match this CEDAR template: https://repo.metadatacenter.org/templates/<template-id>
   [Copy-and-paste the metadata record here]
   ```

## License

This project is licensed under the [MIT License](LICENSE).
