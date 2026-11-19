# PDF Text and Table Extractor

A Python utility for extracting embedded text and automatically detectable tables from PDF files into a UTF-8 text file. It is intended for Persian technical documents, regulations, standards, and engineering references.

## Requirements

- Python 3.10+
- PyMuPDF
- pdfplumber

Install dependencies:

    python -m pip install -r requirements.txt

## Usage

Extract all pages:

    python pdf_to_txt.py "input.pdf" -o "output.txt"

Extract a page range (page numbers are 1-based):

    python pdf_to_txt.py "input.pdf" -o "chapter-5.txt" --pages 105-145

Select multiple ranges/pages:

    python pdf_to_txt.py "input.pdf" -o "selected-pages.txt" --pages 1-5,12,20-24

Choose pages interactively (the program asks which pages to extract):

    python pdf_to_txt.py "input.pdf" -o "selected-pages.txt" --interactive

In interactive mode, enter values such as `8`, `10-25`, or `1-5,12,20-24`.
Press Enter or type `all` to extract the entire PDF. Page numbers start at 1.
Use either `--interactive` or `--pages`, not both.

## Output

For each page, the TXT file contains:
- [PAGE TEXT]: text extracted by PyMuPDF.
- [TABLES]: tables detected by pdfplumber, with cells separated by |.
- A final summary showing processed pages, pages with embedded text, and table count.

The output is UTF-8 with BOM (utf-8-sig), which is convenient for common Windows text editors.

## Limitations and quality control

1. Scanned PDFs: this version does not run OCR. Pages with no embedded text are flagged. OCR with a Persian language model such as Tesseract fas must be added/run separately.
2. Table accuracy: detection is heuristic. Merged cells, borderless tables, multi-line cells, and complex engineering tables may be extracted incorrectly or missed.
3. Persian direction: the script does not reverse Persian strings or apply visual RTL transformations, to avoid corrupting character order. Review with an RTL-capable editor.
4. Reading order: geometric sorting improves extraction but cannot guarantee correct reading order in every multi-column PDF.
5. Verify all extracted numerical/regulatory table values against the original PDF before using them in engineering calculations.

## Recommended workflow

1. Extract the full PDF or the required page range.
2. Review pages flagged as having no embedded text; these may need OCR.
3. Compare every extracted table with its source page.
4. Convert only verified content into structured JSON/CSV with edition, clause, table, and page metadata.


## Upload extracted output directly to GitHub

The script can commit the generated TXT file to a GitHub repository immediately after extraction. It uses the GitHub Contents API and updates an existing file safely by reading its current SHA first.

### 1. Create a GitHub token

Create a fine-grained personal access token with **Contents: Read and write** access to the target repository. Do not put the token in this repository or in the script.

In Windows PowerShell, set it for the current terminal session:

    $env:GITHUB_TOKEN = "YOUR_GITHUB_TOKEN"

### 2. Extract and upload in one command

Example for the relevant pages of Mabhas 16:

    python pdf_to_txt.py "mabhas-16-1402-08.pdf" -o "chapter-5.txt" --pages 107-145 --github-upload

By default, the output is committed to:

    alizarei-eng/mechanical-engineering-library
    references/Mabhas16/extraction-tests/chapter-5.txt

### Optional GitHub settings

Choose another repository:

    --github-repo "owner/repository"

Choose another destination path:

    --github-path "references/Mabhas16/raw/chapter-5.txt"

Choose another branch:

    --github-branch "main"

Set a commit message:

    --github-message "Add chapter 5 extraction"

Complete example:

    python pdf_to_txt.py "mabhas-16-1402-08.pdf" -o "chapter-5.txt" --pages 107-145 --github-upload --github-path "references/Mabhas16/extraction-tests/chapter-5.txt" --github-message "Extract chapter 5 tables"

### Upload behavior and safety

- Without `--github-upload`, the script only creates the local output as before.
- The local TXT file is retained after upload.
- If the destination file does not exist, GitHub creates it; if it exists, the script updates it using its current file SHA.
- The script uploads only the generated TXT output, not the source PDF.
- If upload fails, the local output remains available and the script exits with a non-zero status.
- The token is read from the `GITHUB_TOKEN` environment variable and is never written to the output file.
- Extracted tables are still unverified. Check every value against the original PDF before using it in engineering calculations.
