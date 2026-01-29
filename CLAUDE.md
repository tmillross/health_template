# Claude Workspace Guidelines - Health Data Template

This is a template repository for creating new health data processing projects. Each client gets their own instance of this structure.

## Markdown Formatting

**Heading Numbering:** Do not add arbitrary numbers to markdown headings unless they represent genuinely sequential steps or items (e.g., use `## PII Protection` not `## 1. PII Protection`). Only number headings when they represent actual sequences (e.g., "Step 1", "Phase 2").

## CRITICAL: PII Protection

This repository will contain **sensitive personal health information**.

**NEVER read or access:**
- `contains_pii/0_raw_inbox/` - Raw documents with PII
- `contains_pii/1_extracted_text/` - Extracted text with PII
- `pii_config.json` - Contains actual PII values

**Safe to access:**
- `no_pii/2_unstructured/` and higher - PII-redacted data
- `*.md` documentation files
- Template configuration files (`pii_config.template.json`)

**See `../health_process/PII_PROTECTION_GUIDE.md` for complete details.**

## Repository Structure

```
health_person_name/
├── contains_pii/
│   ├── 0_raw_inbox/
│   │   ├── nhs_gp/             ← NHS SAR / GP records - CONTAINS PII
│   │   └── other/              ← Dental, private, wearables - CONTAINS PII
│   ├── 1_extracted_text/
│   │   ├── nhs_gp/             ← CONTAINS PII - DO NOT ACCESS
│   │   └── other/              ← CONTAINS PII - DO NOT ACCESS
│   └── pii_config.template.json
├── no_pii/
│   ├── 2_unstructured/
│   │   ├── nhs_gp/             ← Safe - PII removed
│   │   └── other/              ← Safe - PII removed
│   ├── 3_structured/           ← Safe - Structured data
│   └── 4_ai_outputs/           ← Safe - Analysis outputs
├── PROCESS_LOG.md
└── pii_config.json             ← CONTAINS PII - DO NOT ACCESS
```

### Source type subfolders

- **`nhs_gp/`** — NHS Subject Access Request outputs, GP records. Processed with `extract_sar.py` (single large PDF, needs OCR).
- **`other/`** — Dental, private providers, wearables, standalone documents. Processed with `extract_batch.py` (multiple simple PDFs, embedded text only).

## File Naming Convention

Use underscores and lower_case, not dashes:
- data_summary.md (correct)
- data-summary.md (incorrect)
- Data Summary.md (incorrect, no spaces)

British English: "organisation" not "organization"

For detailed medical file naming standards, see `../health_process/system_reference/naming_convention.md`

## Python Environment

Always use the shared venv from health_process:
```bash
source ../health_process/venv/bin/activate
```

Repository siblings:
```
/Users/tom/health/
├── health_business/  (business documentation)
├── health_process/   (shared scripts & venv)
├── health_template/  (this template)
└── health_person_*/  (individual client projects)
```

## Processing Pipeline

The pipeline follows numbered stages. Files stay in place after processing — `PROCESS_LOG.md` records what has been done.

1. **Raw Intake** (`contains_pii/0_raw_inbox/nhs_gp/` or `other/`) - Place raw medical documents here
2. **Text Extraction** (`contains_pii/1_extracted_text/nhs_gp/` or `other/`) - Text extracted from documents
3. **PII Redaction** → Data moves to `no_pii/2_unstructured/nhs_gp/` or `other/` after redaction
4. **FHIR Structuring** (`no_pii/3_structured/`) - Use Claude Code with FHIR plugin to convert to FHIR format
5. **AI Analysis** (`no_pii/4_ai_outputs/`) - AI processing and analysis outputs

**After PII removal, use Claude Code with the FHIR plugin to structure clinical health data into FHIR format.** FHIR is ideal for clinical data (NHS records, GP records, lab results, medications, diagnoses). For wearables, genetic data, and other non-clinical data, preserve original formats. See `../health_process/system_reference/fhir_data_compatibility.md` for guidance.

## Processing Scripts

All processing scripts are in `../health_process/scripts/`. See `../health_process/CLAUDE.md` for complete pipeline documentation.

### Quick Commands

**Extract text from NHS SAR PDF (single PDF, with OCR):**
```bash
source ../health_process/venv/bin/activate
python3 scripts/0-1_extract_text/extract_sar.py contains_pii/0_raw_inbox/nhs_gp/sar_document.pdf
```

**Extract text from other PDFs (batch, embedded text only):**
```bash
source ../health_process/venv/bin/activate
python3 scripts/0-1_extract_text/extract_batch.py
```

**Remove PII (single file):**
```bash
source ../health_process/venv/bin/activate
python3 scripts/1-2_remove_pii/strip_pii.py <input_file> <output_dir>
```

**Remove PII (batch, preserves nhs_gp/other/ structure):**
```bash
source ../health_process/venv/bin/activate
python3 scripts/1-2_remove_pii/strip_pii.py --batch contains_pii/1_extracted_text/ no_pii/2_unstructured/
```

**Structure data into FHIR format:**
After PII removal, use Claude Code with the FHIR plugin to structure the cleaned text into standardised FHIR resources. See `../health_process/CLAUDE.md` for detailed FHIR workflow.

## Script Development Guidelines

### Language Preferences

**Always prefer Python for scripts** unless there's a specific reason to use Bash:

- ✅ **Use Python for:** Data processing, file manipulation, text parsing, API calls, complex logic
- ⚠️ **Use Bash only for:** Simple system commands, environment setup, git operations

**Avoid writing `.sh` shell scripts when Python is sufficient.** Python provides better error handling, readability, and maintainability.

### Tool Selection

**Use the FHIR plugin (`fhir-developer` skill) when appropriate:**

- Creating or fixing FHIR resources interactively
- Converting unstructured clinical text to FHIR format
- Getting guidance on FHIR R4 compliance
- Working through validation errors

**Use Python scripts for:**

- Batch processing and automation
- Validation of multiple files
- Repeatable operations

**FHIR structuring guide:** See `scripts/2-3_structure_fhir/FHIR_STRUCTURING_GUIDE.md` for comprehensive LLM instructions on FHIR structuring, including extraction targets, R4 compliance rules, and decisions log.

**FHIR validation:** Use `python3 scripts/2-3_structure_fhir/validate_fhir.py no_pii/3_structured/` to validate FHIR files with the `fhir.resources` library.

## Logging

Log **every** pipeline step to `PROCESS_LOG.md` at the repository root. Files stay in place after processing — `PROCESS_LOG.md` is the sole record of what has been processed and when.

All Python scripts log automatically via the shared `scripts/utils/log_utils.py` utility. The format includes the command used for reproducibility:

```markdown
## YYYY-MM-DD

- **Text extraction**: `file.pdf` → `1_extracted_text/other/output.md`
  - Command: `python3 scripts/0-1_extract_text/extract_batch.py`
- **PII removal (batch)**: `contains_pii/1_extracted_text/` → `no_pii/2_unstructured/`
  - Command: `python3 scripts/1-2_remove_pii/strip_pii.py --batch contains_pii/1_extracted_text/ no_pii/2_unstructured/`
  - Result: 5 files processed, 42 total redactions
- **FHIR structuring (LLM)**: `no_pii/2_unstructured/` → `no_pii/3_structured/`
  - Files: patient.json, conditions.json, medications_current.json, ...
- **FHIR validation**: `no_pii/3_structured/` (10 files)
  - Command: `python3 scripts/2-3_structure_fhir/validate_fhir.py no_pii/3_structured/ no_pii/3_structured/fhir_validation_report.md`
  - Result: 6 passed, 4 failed
```

### FHIR structuring (LLM step)

The FHIR structuring step (stage 2→3) is performed interactively by Claude Code, not by a script. **After completing FHIR structuring, always log the action to `PROCESS_LOG.md`** with:

- **Action**: "FHIR structuring (LLM)"
- **Detail**: Source and destination directories
- **Files**: List of FHIR resources created or updated

## References

- **Detailed processing guide:** `../health_process/CLAUDE.md`
- **PII protection:** `../health_process/PII_PROTECTION_GUIDE.md`
- **File naming standards:** `../health_process/system_reference/naming_convention.md`
- **SAR structuring prompt:** `../health_process/system_reference/SAR_output_explainer_prompt.md`