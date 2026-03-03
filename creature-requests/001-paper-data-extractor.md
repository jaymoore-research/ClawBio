# [Skill] Paper Data Extractor

**Labels**: help wanted

## Skill name

`paper-data-extractor`

## What it does

Extracts, formats, and structures raw data from scientific publications — including supplementary materials, figures, tables, and inline text. Embeds images for visual reference, links to open-access repositories (Zenodo, NCBI), and flags whether source data and code are openly available. Turns a PDF/DOI into a machine-readable, citation-ready data package.

## Bioinformatics tools it wraps

- **PDF/XML parsing**: PyMuPDF (fitz), pdfplumber, lxml (for PubMed Central JATS XML)
- **Figure data extraction**: Plot digitiser logic (contour detection, OCR via Tesseract/EasyOCR, axis calibration) to recover numerical data from graphs
- **Table extraction**: Camelot, tabula-py for PDF tables; pandas for supplementary CSV/Excel/TSV
- **Image embedding**: Extracts and embeds raster figures (PNG/SVG) from PDFs with captions
- **Repository linking**: NCBI Entrez API (PubMed, GEO, SRA, BioProject), Zenodo REST API, DataCite API
- **Open-access detection**: Unpaywall API, DOAJ, PubMed Central OA subset
- **LLM summarisation**: Uses the active agent model to interpret text, captions, and methods sections

## Input / Output

- **Input**: DOI, PMID, PubMed Central ID (PMCID), PDF file path, or URL to a paper
- **Output**:
  - `extracted_data/` — structured CSVs/JSONs recovered from tables, figures, and text
  - `figures/` — embedded images (PNG) with extracted captions
  - `supplementary/` — downloaded and parsed supplementary files
  - `report.md` — structured summary with:
    - Paper metadata (title, authors, journal, date, DOI)
    - Data availability statement (verbatim)
    - Extracted tables (markdown formatted)
    - Figure descriptions and digitised data points
    - Links to repositories (GEO, SRA, Zenodo, GitHub, etc.)
    - Open-access status and licence
  - `links.json` — machine-readable list of all repository/data links found
  - `commands.sh` + `checksums.sha256` — reproducibility bundle

## Workflow

### 1. Paper acquisition
- Accept DOI, PMID, PMCID, URL, or local PDF
- If DOI/PMID: resolve to full text via PubMed Central OA, Unpaywall, or Zenodo
- Download PDF and/or JATS XML if available
- Warn if paper is paywalled and no open-access version exists

### 2. Metadata extraction
- Parse title, authors, abstract, journal, publication date, DOI
- Extract data availability statement (usually in Methods or end matter)
- Identify licence (CC-BY, CC-BY-NC, etc.)

### 3. Supplementary data retrieval
- Follow supplementary material links from the publisher or PMC
- Download all supplementary files (Excel, CSV, PDF appendices, ZIP archives)
- Parse tabular supplementary data into structured CSVs

### 4. Table extraction
- Detect and extract tables from PDF body using Camelot/tabula-py
- Parse JATS XML `<table-wrap>` elements if available
- Clean headers, merge multi-line cells, infer column types
- Output as CSV + markdown

### 5. Figure data extraction
- Extract embedded images from PDF
- For line charts, bar charts, scatter plots: apply plot digitisation
  - Detect axes, tick marks, labels via OCR
  - Calibrate coordinate system from axis labels
  - Extract data points (pixel → data coordinates)
  - Output as CSV with columns matching axis labels
- For non-quantitative figures (schematics, photos): embed image + caption only
- Flag figures where digitisation confidence is low

### 6. Text data extraction
- Use LLM to identify numerical results reported inline (e.g., "p = 0.003", "HR = 1.42, 95% CI 1.1-1.8")
- Extract sample sizes, cohort descriptions, effect sizes
- Structure as key-value pairs in JSON

### 7. Repository and open-source linking
- Search for GEO accession numbers (GSE/GSM/GPL patterns)
- Search for SRA/BioProject/BioSample accessions
- Search for Zenodo DOIs (10.5281/zenodo.*)
- Search for GitHub/GitLab repository URLs
- Search for Data Availability / Code Availability sections
- Validate links are live (HTTP HEAD check)
- Classify each link: raw data, processed data, code, protocol, other

### 8. Open-access assessment
- Query Unpaywall API for OA status
- Check DOAJ for journal OA policy
- Report: full OA, green OA (preprint), bronze, or closed
- Link to best available open version (PMC, preprint, Zenodo)

### 9. Report generation
- Assemble `report.md` with all extracted data, figures, links
- Generate `links.json` for programmatic consumption
- Create reproducibility bundle (`commands.sh`, `checksums.sha256`)
- Log all actions to `analysis_log.md`

## Example queries

- "Extract all data from this paper: doi:10.1038/s41588-023-01540-6"
- "Get the raw data from Figure 3 of this PDF" (with file attached)
- "Find the GEO dataset and supplementary tables for PMID 38347890"
- "Is the data for this paper openly available? doi:10.1016/j.cell.2024.01.029"
- "Download and parse all supplementary files from PMC10234567"
- "Digitise the survival curve in Figure 2A of this PDF"

## Why it matters

Researchers spend enormous time manually extracting data from papers — copying tables from PDFs, reading numbers off graphs, hunting for supplementary files, and tracking down whether raw data was deposited anywhere. This is especially painful for:

- **Meta-analyses and systematic reviews**: Need structured data from dozens/hundreds of papers
- **Reproducibility checks**: Need to verify claims against deposited data
- **Data integration**: Need to combine results across studies into unified datasets
- **Open science audits**: Need to assess whether papers actually make data available as claimed

No existing ClawBio skill handles this. The lit-synthesizer finds and summarises papers but doesn't extract their data. This skill fills that gap — it turns a paper reference into a structured, machine-readable data package.

## Dependencies

```yaml
metadata:
  openclaw:
    requires:
      bins:
        - python3
        - tesseract
      env: []
      config: []
    always: false
    emoji: "🦖"
    os: [macos, linux]
    install:
      - kind: uv
        package: pymupdf
      - kind: uv
        package: pdfplumber
      - kind: uv
        package: camelot-py
      - kind: uv
        package: tabula-py
      - kind: uv
        package: pandas
      - kind: uv
        package: easyocr
      - kind: uv
        package: biopython
      - kind: uv
        package: httpx
      - kind: uv
        package: openpyxl
      - kind: uv
        package: lxml
      - kind: uv
        package: Pillow
```

## Composability

- **Chains with lit-synthesizer**: lit-synthesizer finds relevant papers, paper-data-extractor pulls the data out of them
- **Chains with vcf-annotator**: Extract variant tables from papers, feed into annotation
- **Chains with equity-scorer**: Extract population-level data from papers for equity analysis
- **Chains with bio-orchestrator**: Auto-route "get data from this paper" queries

## Are you willing to implement it?

- [ ] Yes, I'd like to build this skill
- [ ] No, I'm proposing it for someone else to pick up
