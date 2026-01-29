# Health Data Processing Template

This is a health data processing pipeline designed to safely handle sensitive medical documents by separating PII-containing data from anonymised data.

## Folder Structure

### contains_pii/ - Data with Personally Identifiable Information

This directory contains all data that includes PII and should be treated as sensitive.

```
contains_pii/
├── 0_raw_inbox/
│   ├── nhs_gp/             NHS SAR outputs, GP records
│   └── other/              Dental, private, wearables, standalone docs
├── 1_extracted_text/
│   ├── nhs_gp/             Extracted text from NHS/GP documents
│   └── other/              Extracted text from other documents
└── pii_config.template.json  Template for PII redaction configuration
```

### no_pii/ - Anonymised Data

This directory contains data with all PII removed and is safe for broader analysis.

```
no_pii/
├── 2_unstructured/
│   ├── nhs_gp/             Anonymised NHS/GP text
│   └── other/              Anonymised other text
├── 3_structured/           Structured, anonymised data
└── 4_ai_outputs/           AI processing results and analyses
```

### Source type subfolders

- **`nhs_gp/`** — NHS Subject Access Request outputs, GP records. These are typically large single PDFs requiring OCR and semi-supervised extraction.
- **`other/`** — Dental records, private provider documents, wearable data exports, standalone reports. Typically multiple smaller PDFs with embedded text.

## Processing Pipeline

The pipeline follows a numbered sequence. Files stay in place after processing — `PROCESS_LOG.md` records what has been done and when.

1. **Raw Intake** (`contains_pii/0_raw_inbox/nhs_gp/` or `other/`) - Place raw medical documents here
2. **Text Extraction** (`contains_pii/1_extracted_text/nhs_gp/` or `other/`) - Text extracted from documents
3. **PII Redaction** → Data moves to `no_pii/2_unstructured/nhs_gp/` or `other/` after redaction
4. **Structuring** (`no_pii/3_structured/`) - Convert unstructured text to structured FHIR format
5. **AI Analysis** (`no_pii/4_ai_outputs/`) - AI processing and analysis outputs

### Two extraction workflows

- **NHS SAR extraction** (`extract_sar.py`): Processes a single large PDF from `nhs_gp/` using embedded text + OCR, with watermark removal and intelligent source selection.
- **Batch extraction** (`extract_batch.py`): Extracts embedded text from all PDFs in `other/`. No OCR needed.

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
