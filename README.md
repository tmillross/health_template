# Health Data Processing Template

This is a health data processing pipeline designed to safely handle sensitive medical documents by separating PII-containing data from anonymised data.

## Folder Structure

### contains_pii/ - Data with Personally Identifiable Information

This directory contains all data that includes PII and should be treated as sensitive.

```
contains_pii/
├── 0_raw_inbox/
│   ├── 0_to_process/      Raw documents awaiting processing
│   └── 1_processed/       Raw documents that have been processed
├── 1_extracted_text/
│   ├── 0_to_process/      Extracted text awaiting PII redaction
│   └── 1_processed/       Extracted text that has been redacted
└── pii_config.template.json  Template for PII redaction configuration
```

### no_pii/ - Anonymised Data

This directory contains data with all PII removed and is safe for broader analysis.

```
no_pii/
├── 2_unstructured/
│   ├── 0_to_process/      Anonymised text awaiting structuring
│   └── 1_processed/       Text that has been structured
├── 3_structured/          Structured, anonymised data
└── 4_ai_outputs/          AI processing results and analyses
```

## Processing Pipeline

The pipeline follows a numbered sequence:

1. **Raw Intake** (`contains_pii/0_raw_inbox/0_to_process/`) - Place raw medical documents here
2. **Text Extraction** (`contains_pii/1_extracted_text/0_to_process/`) - Text extracted from documents
3. **PII Redaction** → Data moves to `no_pii/2_unstructured/0_to_process/` after redaction
4. **Structuring** (`no_pii/3_structured/`) - Convert unstructured text to structured data
5. **AI Analysis** (`no_pii/4_ai_outputs/`) - AI processing and analysis outputs

In the folders: `0_raw_inbox`, `1_extracted_text`, `2_unstructured` there should be no files in the root, only in the 2 subfolders: `0_to_process` and `1_processed`. 

## File movement, creation & logging

After a file has been processed (e.g. from 0->1 or 2->3), the original file should be moved from `0_to_process` to `1_processed`.
The repo root file: PROCESS_LOG.md should be used for documenting all file movements and creation, including relative filepaths from the root.

## PII Configuration

The `pii_config.template.json` file provides a template for configuring PII redaction. Create a copy named `pii_config.json` (which should be gitignored) and replace placeholder values with actual PII to redact:

- **Names** - Full names and surnames
- **Postcodes** - UK postcode formats
- **Phone numbers** - Extracts final 10 digits (handles both UK and international formats)
- **Email addresses**
- **Physical addresses** - Street names, building names
- **Date of birth**
- **NHS numbers** (optional removal)
- **Hospital IDs** (optional removal)
- **Dates** (optional removal)

## Security Notes

- The `contains_pii/` directory should never be committed to version control or shared
- Only share data from the `no_pii/` directory after verification
- Create `pii_config.json` from the template and keep it secure
- The template uses British English spellings and conventions
