# Claude Workspace Guidelines

## Data Processing Approach

When the user presents a simple data processing task, prefer direct LLM-based processing over writing code, unless:

1. **The task requires code** - The user explicitly asks for a script, tool, or automation
2. **The data is too large** - The dataset exceeds what can be reasonably processed in conversation
3. **Complex operations required** - The task involves calculations, API calls, or complex transformations better suited to code

The goal is to leverage LLM capabilities for most one-off tasks while reserving code for automation, complexity, and scale.

# File and folder naming convention
Use underscores and lower_case, not dashes.
NEVER create or write files with spaces in filenames
Replace spaces with underscores when creating files

# Writing style
British English, e.g. organisation not organization.

# Data processing pipeline

## Folder structure pattern
Each processing stage uses `0_to_process/` for pending items and `1_processed/` for completed originals. This allows tracking what has been processed while preserving source files.

## Pipeline stages

1. **Raw Intake** (`contains_pii/0_raw_inbox/0_to_process/`) - Place raw medical documents here
2. **Text Extraction** (`contains_pii/1_extracted_text/0_to_process/`) - Markdown extracted from documents
   - Run: `python3 scripts/0-1_process_inbox/convert_pdfs_to_md.py`
   - Uses PDF table of contents for section headers
   - Processed PDFs move to `contains_pii/0_raw_inbox/1_processed/`
3. **PII Redaction** → Data moves to `no_pii/2_unstructured/0_to_process/` after redaction
4. **Structuring** (`no_pii/3_structured/`) - Convert unstructured text to structured data
5. **AI Analysis** (`no_pii/4_ai_outputs/`) - AI processing and analysis outputs

## Logging
Log file processing actions to `PROCESS_LOG.md` with date, action taken, and files affected.