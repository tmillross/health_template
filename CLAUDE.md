# Claude Workspace Guidelines - Health Data Template

This is a template repository for creating new health data processing projects. Each client gets their own instance of this structure.

## Markdown Formatting

**Heading Numbering:** Do not add arbitrary numbers to markdown headings unless they represent genuinely sequential steps or items (e.g., use `## PII Protection` not `## 1. PII Protection`). Only number headings when they represent actual sequences (e.g., "Step 1", "Phase 2").

## ⚠️ CRITICAL: PII Protection

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
│   │   ├── 0_to_process/     ← CONTAINS PII - DO NOT ACCESS
│   │   └── 1_processed/      ← CONTAINS PII - DO NOT ACCESS
│   ├── 1_extracted_text/
│   │   ├── 0_to_process/     ← CONTAINS PII - DO NOT ACCESS
│   │   └── 1_processed/      ← CONTAINS PII - DO NOT ACCESS
│   └── pii_config.template.json
├── no_pii/
│   ├── 2_unstructured/
│   │   ├── 0_to_process/     ← Safe - PII removed
│   │   └── 1_processed/      ← Safe - PII removed
│   ├── 3_structured/         ← Safe - Structured data
│   └── 4_ai_outputs/         ← Safe - Analysis outputs
├── PROCESS_LOG.md
└── pii_config.json           ← CONTAINS PII - DO NOT ACCESS
```

## File Naming Convention

Use underscores and lower_case, not dashes:
- ✓ `data_summary.md`
- ✗ `data-summary.md`
- ✗ `Data Summary.md` (no spaces)

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

The pipeline follows numbered stages:

1. **Raw Intake** (`contains_pii/0_raw_inbox/0_to_process/`) - Place raw medical documents here
2. **Text Extraction** (`contains_pii/1_extracted_text/0_to_process/`) - Text extracted from documents
3. **PII Redaction** → Data moves to `no_pii/2_unstructured/0_to_process/` after redaction
4. **Structuring** (`no_pii/3_structured/`) - Convert unstructured text to structured data
5. **AI Analysis** (`no_pii/4_ai_outputs/`) - AI processing and analysis outputs

## Processing Scripts

All processing scripts are in `../health_process/scripts/`. See `../health_process/CLAUDE.md` for:
- PII removal scripts
- PDF to markdown conversion
- Text cleanup tools
- Complete pipeline documentation

### Quick Commands

**Extract text from PDFs:**
```bash
source ../health_process/venv/bin/activate
python3 ../health_process/scripts/0-1_process_inbox/convert_pdfs_to_md.py
```

**Remove PII:**
```bash
source ../health_process/venv/bin/activate
cd ../health_process/scripts/1-2_remove_pii/
./strip_pii.sh
```

## Logging

Log all processing actions to `PROCESS_LOG.md` at the repository root. This file starts empty in the template and is filled as processing begins for a specific client/person.

Format:
```markdown
## YYYY-MM-DD

**Action:** [What was done]
**Files:** [Files affected]
**Result:** [Outcome]
```

## File Movement

After processing:
1. Output files go to next stage's `0_to_process/` folder
2. Original files move from `0_to_process/` to `1_processed/`
3. Log the movement in `PROCESS_LOG.md`

## References

- **Detailed processing guide:** `../health_process/CLAUDE.md`
- **PII protection:** `../health_process/PII_PROTECTION_GUIDE.md`
- **File naming standards:** `../health_process/system_reference/naming_convention.md`
- **SAR structuring prompt:** `../health_process/system_reference/SAR_output_explainer_prompt.md`
