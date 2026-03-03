# CLAUDE.md

This is a template repository for creating new health data processing projects. Each client gets their own instance of this structure. Processing scripts are symlinked from `../health_process/scripts/`.

## Markdown Formatting

**Heading Numbering:** Do not add arbitrary numbers to markdown headings unless they represent genuinely sequential steps or items. Only number headings when they represent actual sequences (e.g., "Step 1", "Phase 2").

## CRITICAL: PII Protection

This repository will contain **sensitive personal health information**.

**NEVER read or access:**

- `contains_pii/0_raw_inbox/` — Raw documents with PII
- `contains_pii/1_extracted_text/` — Extracted text with PII
- `pii_config.json` — Contains actual PII values

**Safe to access:**

- `no_pii/2_unstructured/` and higher — PII-redacted data
- `*.md` documentation files
- Template configuration files (`*.template.json`)

## File Naming Convention

Use underscores and lower_case, not dashes:

- data_summary.md (correct)
- data-summary.md (incorrect)

British English: "organisation" not "organization"

For detailed medical file naming standards, see `../health_process/system_reference/naming_convention.md`

## Script Development Guidelines

**Always prefer Python** over Bash scripts. Python provides better error handling, readability, and maintainability.

See `../health_process/CLAUDE.md` for complete script reference and development guidelines.

## Logging

Log **every** pipeline step to `PROCESS_LOG.md`. Python scripts log automatically via `scripts/utils/log_utils.py`. For interactive steps (FHIR structuring with Claude Code), log manually with:

- **Action**: "FHIR structuring (LLM)"
- **Detail**: Source and destination directories
- **Files**: List of FHIR resources created or updated

See `README.md` for project structure, configuration, pipeline commands, and references.
