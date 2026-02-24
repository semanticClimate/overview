# semanticClimate Projects: Specification

**Date:** 2026-02-24 (system date of generation)  
**Scope:** Top-level overview repo for all semanticClimate projects; includes libraries from petermr (see README).  
**Conventions:** amilib style guide (`../amilib/docs/style_guide_compliance.md`), pygetpapers styleguide (`../pygetpapers/docs/styleguide.md`); absolute imports; no `sys.path`; system date where relevant.

---

## 1. Project inventory and functionality

### 1.1 pygetpapers (petermr/pygetpapers)

**Role:** Download papers and metadata from open-access repositories.

**Primary functionality:**
- Query and download from Europe PMC, bioRxiv, medRxiv, Crossref, OpenAlex, Redalyc, SciELO, Upspace.
- Output: timestamped directories under `~/pygetpapers/` (or custom path) with per-article folders.
- Optional: XML fulltext, PDF, HTML; CSV of results; DataTables HTML + JSON for interactive browsing.

**Primary inputs:**
- Query string, API/repository choice, limit, date ranges, flags (e.g. `--xml`, `--pdf`, `--makecsv`, `--datatables`).

**Primary outputs:**
- **Directories:** `{output_root}/{repo}_{timestamp}/` with `{paper_id}/` subdirs.
- **Files:** `fulltext.xml`, `fulltext.pdf`, `fulltext.html`, `fulltext.pdf.html` (when requested); `eupmc_result.json` / `*_result.json` per article; `eupmc_results.json` (or repo-specific) at project level; `*_papers_data.json` for DataTables; `datatables.html`; CSV when `--makecsv`.

**Main file types for transfer:** `.xml`, `.pdf`, `.html`, `.json`, `.csv`.

---

### 1.2 amilib (petermr/amilib)

**Role:** Library for document download, transformation, and analysis (Open Access, IPCC, UNFCCC, etc.). Used by other projects; entry points via CLI; longer-term integration with docanalysis and pygetpapers.

**Primary functionality:**
- **DICT:** Create/edit/validate AMI dictionaries (words → dictionary XML/HTML), optional Wikipedia/Wikidata/Wiktionary descriptions.
- **HTML:** Create, transform, annotate HTML (e.g. dictionary-based annotation).
- **PDF:** Convert PDF → HTML (pdfminer/pdfplumber), optional images, page ranges, clipping.
- **SEARCH:** Search and annotate HTML, build indexes (annotate, index, no_input_styles).

**Primary inputs:**
- DICT: word lists, existing dictionary files, description sources.
- HTML: HTML files, dictionary for annotation.
- PDF: PDF files or paths.
- SEARCH: HTML paths, dictionary (for annotate).

**Primary outputs:**
- DICT: `.xml` or `.html` dictionaries.
- HTML: annotated or transformed HTML.
- PDF: HTML (and optionally images) from PDF.
- SEARCH: annotated HTML, indexes.

**Main file types for transfer:** `.xml`, `.html`, `.pdf`, dictionary formats (AMI XML/HTML).

---

### 1.3 docanalysis (petermr/docanalysis)

**Role:** CLI that ingests CProjects (corpora) and runs text analysis: sectioning, NLP/entity extraction, dictionary generation. Uses NLTK, spaCy/scispaCy; can invoke pygetpapers to create CProjects.

**Primary functionality:**
- **--run_pygetpapers:** Download papers from Europe PMC into a CProject (creates CTree per article with `fulltext.xml`, `eupmc_result.json`).
- **--make_section:** Section articles from `fulltext.xml` (JATS) into `sections/` directory trees.
- Entity extraction (spaCy/scispaCy): entities (e.g. ORG, PERSON) from sections.
- Dictionary search over sections; output CSV/HTML/JSON.
- **--make_ami_dict:** Build AMI dictionary from extracted entities.
- **--extract_abb:** Abbreviation/expansion extraction → AMI dictionary.
- **--search_html:** Search HTML documents (e.g. IPCC).

**Primary inputs:**
- CProject directory (from pygetpapers or pre-existing) containing CTrees with `fulltext.xml` (and optionally `eupmc_result.json`).
- Dictionary file/URL for annotation or supervised extraction.
- Flags: section names, entity types, spacy model, output paths.

**Primary outputs:**
- CProject with added `sections/` per CTree (sectioned XML).
- CSV/HTML/JSON of sentences/terms/entities.
- AMI dictionary XML (entities, abbreviations).

**Main file types for transfer:** `.xml` (fulltext, sectioned), `.json` (eupmc_result), `.csv`, `.html`, `.json` (output), AMI `.xml` dictionaries.

---

### 1.4 txt2phrases

**Role:** Pipeline from documents (PDF/HTML) to plain text and then to keyphrases; can consume pygetpapers output.

**Primary functionality:**
- **pdf2txt:** PDF → plain text.
- **html2txt:** HTML → plain text.
- **keyphrases:** Extract keyphrases from text using NLP models (e.g. Hugging Face); optional TF-IDF classification (specific/general).
- **auto:** Full pipeline on a directory (detects pygetpapers-style layout): PDF/HTML → text → keyphrases; batch processing.

**Primary inputs:**
- Single file or directory (PDF, HTML, or text).
- For `auto`: pygetpapers-style directory (e.g. `{paper_id}/fulltext.pdf`, `fulltext.html`).

**Primary outputs:**
- Plain text files; keyword/keyphrase CSVs (e.g. `*_keywords.csv`).

**Main file types for transfer:** `.pdf`, `.html`, `.txt`, `.csv`.

---

### 1.5 semantic_corpus

**Role:** Create and manage personal scientific corpora: search, download, organize; optional BAGIT preservation.

**Primary functionality:**
- **create:** Initialize corpus directory.
- **search:** Search Europe PMC / arXiv; save results to JSON.
- **download:** Search + download in specified formats (e.g. pdf, xml).
- **Ingestion:** Ingest pygetpapers-style directories (e.g. `PMC*/` with `eupmc_result.json`, `fulltext.xml`) into BAGIT corpus; copies XML/PDF and updates manifest.

**Primary inputs:**
- Query, repository choice, limit, formats, output path; optional YAML config.
- For ingestion: path to pygetpapers output directory (e.g. from amilib test resources or real runs).

**Primary outputs:**
- Corpus directory (optionally BAGIT); JSON search results; downloaded files (PDF, XML) in corpus structure.

**Main file types for transfer:** `.json` (metadata, search results), `.xml`, `.pdf`; BAGIT manifest and structure.

---

### 1.6 encyclopedia

**Role:** Extract and analyse keywords from scientific documents (e.g. climate, IPCC); store in dictionaries; browse via web UI.

**Primary functionality:**
- **Keyword_extraction:** NLP-based keyword extraction from text (e.g. Hugging Face); batch; CSV output.
- **Dictionary:** Structured storage of chapters, keywords, full text; HTML/text versions; CSV export.
- **Encyclopedia browser:** Streamlit app to search/browse entries (Whoosh, NLTK); up to ~5k entries.

**Primary inputs:**
- Text/documents (e.g. IPCC chapters); word lists or existing dictionary content.
- Depends on **amilib** for HTML/dictionary/Wikimedia utilities (HtmlLib, AmiDictionary, WikipediaPage, etc.).

**Primary outputs:**
- Keyword CSVs; dictionary structures (HTML, text); browser UI (Streamlit).

**Main file types for transfer:** `.csv`, `.html`, plain text; AMI-style dictionary data; corpus can be built using pygetpapers (see demo notebook).

---

### 1.7 pyamiimage (petermr/pyamiimage)

**Role:** Extract semantic information from scientific diagrams (e.g. terpene synthase pathways); pixel → annotated “smart” diagram.

**Primary functionality:**
- Image manipulation (e.g. grayscale); arrow/graph extraction (AmiGraph); OCR (Tesseract via AmiOCR).
- Output: annotated image (substrates, products, enzymes); future: CML/GPML support.

**Primary inputs:**
- Image files (scientific figures/diagrams).

**Primary outputs:**
- Annotated images; (future) CML/GPML.

**Main file types for transfer:** Image formats (e.g. PNG, JPEG); (future) `.cml`, GPML.

---

### 1.8 overview (this repo)

**Role:** Top-level overview and interoperability documentation for semanticClimate projects; references petermr and other project repos.

**Primary functionality:** Documentation and specifications; no runtime I/O beyond docs and diagrams.

---

## 2. How the projects interact

- **pygetpapers** produces **CProject-like** output (per-article dirs with `fulltext.xml`, `*_result.json`). This is the main **ingestion source** for:
  - **docanalysis:** `--run_pygetpapers` creates CProject; docanalysis then sections and analyses.
  - **semantic_corpus:** ingests pygetpapers directories into BAGIT corpora (copies `fulltext.xml`, `fulltext.pdf`, normalizes metadata).
  - **txt2phrases:** `auto` pipeline detects pygetpapers layout and runs pdf2txt/html2txt + keyphrases on it.

- **docanalysis** consumes **CProject** (fulltext.xml, sections) and produces **AMI dictionaries** (XML) and CSV/HTML/JSON. Those dictionaries can be used by:
  - **amilib** (DICT, HTML annotation, SEARCH) for annotation and search.

- **amilib** is a **shared library** used by:
  - **encyclopedia** (HtmlLib, AmiDictionary, Wikimedia, etc.).
  - Other projects for PDF→HTML, HTML annotation, dictionaries.

- **encyclopedia** uses **amilib** and can use **pygetpapers** for corpus creation (e.g. demo notebook).

- **txt2phrases** is **downstream** of pygetpapers (and optionally amilib for test PDFs); its CSV/keyphrases can feed into keyword lists or dictionaries used elsewhere.

- **pyamiimage** is **standalone** for diagram extraction; no direct file interchange with the others in this spec (future CML/GPML could align with chemistry/pathway tooling).

---

## 3. Main file types used for information transfer

| File type | Use |
|-----------|-----|
| **.xml** | Full-text articles (JATS); sectioned XML; AMI dictionaries; (future) CML. |
| **.json** | Repository metadata (`eupmc_result.json`, `*_papers_data.json`); search results; DataTables data. |
| **.html** | Fulltext HTML; annotated HTML; DataTables UI; dictionary HTML; encyclopedia content. |
| **.pdf** | Original papers; amilib PDF→HTML input; corpus storage. |
| **.csv** | Query results; keyword/keyphrase output (txt2phrases, encyclopedia); entity/sentence tables (docanalysis). |
| **.txt** | Plain text from PDF/HTML (txt2phrases); intermediate for keyphrase extraction. |
| **YAML** | semantic_corpus configuration (queries, repositories, formats). |
| **BAGIT** | semantic_corpus corpus layout and manifest. |

---

## 4. Diagram

See `projects_interaction.gv` (Graphviz). Render with:

```bash
dot -Tsvg projects_interaction.gv -o projects_interaction.svg
dot -Tpdf projects_interaction.gv -o projects_interaction.pdf
```
