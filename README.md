# IRR Calculator
**Inter-Rater Reliability Calculator**  
Learning Data Insights · GenAI Evidence Hub

---

## Overview

The IRR Calculator is a self-contained, browser-based desktop application for computing inter-rater reliability (IRR) on systematic literature review coding sheets. It requires no installation, no server, and no internet connection after the initial page load — simply open `irr_calculator.html` in any modern browser.

It was built specifically for the GenAI Evidence Hub project and supports the coding sheet formats used across the three evidence domains: Automated Scoring, Item Generation, and Formative Feedback (upcoming).

---

## Getting Started

1. Download `irr_calculator.html`
2. Open it in Chrome, Firefox, Edge, or Safari
3. Upload your combined coding sheet (see Supported Formats below)
4. Follow the five-step sidebar workflow
5. Click **Calculate IRR** to view results

No Python, Node.js, or package installation required.

---

## Supported File Formats

| Format | Extensions |
|--------|------------|
| Comma-separated | `.csv` |
| Excel | `.xlsx`, `.xls` |
| Tab-separated | `.tsv` |
| Plaintext | `.txt` |

The delimiter is auto-detected for text-based files. Excel files use the first sheet by default.

---

## Coding Sheet Requirements

### Expected Structure
The application expects a **long-format** file where each row represents one reviewer's coding of one research question:

| Paper ID | Research Question ID | Reviewer | Domain | Metric1-type | ... |
|---|---|---|---|---|---|
| 249 | 249.1 | Alice | Education | QWK | ... |
| 249 | 249.1 | Bob | Education | Pearson | ... |
| 249 | 249.2 | Alice | NLP | Accuracy | ... |

### Required Columns
- **`Research Question ID`** — unique identifier per research question (e.g. `249.1`, `249.2`). Auto-selected as the primary item ID column.
- **`Paper ID`** — numeric paper identifier used to group research questions into papers for the summary display (e.g. `249`).
- **`Paper Title`** — title of the paper, used as a display label in the By Paper ID summary cards.
- **`Reviewer`** — coder identifier. Auto-selected as the Coder ID column.
- **`Research Question`** — full text of the research question. Used as a display label in the RQ-level table and in exports.

### Known Artifact Row
The second row of each coding sheet contains the label `"Numerical (index identifier)"` in the Research Question ID column. This row is automatically detected and excluded from all calculations and display.

---

## Coding Sheet Versions

The application supports three coding sheet versions. It auto-detects the version based on signature columns present in the uploaded file, with a confirmation banner and option to override manually.

| Version | Status | Detection Columns |
|---|---|---|
| **Automated Scoring** | Active | `Metric1-type`, `Bias / Fairness Evaluated` |
| **Item Generation** | Active | `Metric1-analysis type`, `Bias / Equity Evaluated` |
| **Formative Feedback** | Placeholder (coming soon) | — |

Upon detecting a version, all IRR-relevant columns for that version are pre-selected in the column picker. Metric columns (Metric1 through Metric10) are only pre-checked if at least one reviewer has entered data for that column.

---

## Supported IRR Metrics

| Metric | Requirements | Data Types |
|---|---|---|
| **Krippendorff's Alpha** | 2+ coders | Nominal, Ordinal, Interval |
| **Pairwise % Agreement** | 2+ coders | All (exact match) |
| **Cohen's Kappa** | Exactly 2 coders | Nominal |
| **Fleiss' Kappa** | 3+ coders | Nominal |

The application validates whether the selected metric is compatible with the data before running and surfaces a clear error if not (e.g., Cohen's Kappa blocked when more than 2 coders are present).

Each column can be assigned a data type — **nominal**, **ordinal**, or **interval** — which affects how Krippendorff's Alpha computes disagreement. Data types are auto-detected on upload and can be changed per column in the column picker.

---

## Calculation Methodology

### Krippendorff's Alpha
Computed using the **coincidence-matrix method** (Hayes & Krippendorff, 2007). This implementation has been validated against the `krippendorff` Python package and produces equivalent results. The delta function uses nominal distance (0 if equal, 1 if not) by default, or ordinal/interval distance when the data type is set accordingly.

### How IRR is Aggregated
- **Per-variable IRR:** Alpha is computed across all coders for each coding column individually.
- **Per-RQ IRR:** The average of all per-variable Alpha values for that Research Question ID.
- **Per-paper IRR:** The average of all per-RQ IRR values for research questions belonging to that Paper ID (e.g. Paper 249 with RQs 249.1 and 249.2 shows the average of both RQ scores).
- **Overall IRR:** The average of all per-RQ values across all papers.
- **Empty cells** are treated as missing and excluded from all calculations.

### A Note on Expected Values
With 9+ coders and genuine variation across coding decisions, Krippendorff's Alpha values in the 0.60–0.70 range are expected and appropriate. Values calculated on earlier data snapshots (with fewer coders) will naturally differ from the current full-coder result.

---

## Results Display

Results are organized into three sections:

### 1. Overall Summary
A side-by-side panel showing:
- **Left:** Overall average IRR across all research questions, with a strength interpretation (Strong >= 0.80, Moderate >= 0.60, Low < 0.60)
- **Right — By Paper ID:** One card per paper, horizontally scrollable, showing:
  - `Paper [ID]` as the heading (e.g. `Paper 249`)
  - Truncated paper title beneath (full title on hover)
  - Color-coded combined IRR score averaged across all RQs for that paper

### 2. Research Question-Level IRR Table
One row per Research Question ID with IRR score(s) and the truncated research question text as a label. Click any row to expand an inline variable-level breakdown showing IRR for each individual coding column for that research question.

### 3. Pairwise Agreement Detail (collapsible)
Appears when Pairwise % Agreement is selected. Shows agreement per coder pair per column, collapsed by default.

---

## Exporting Results

Click **Export CSV** or **Export Excel** in the results header. Both exports mirror the on-screen layout with two sections:

**Overall Summary**
- One row showing the overall average IRR per active metric

**By Paper ID**
- One row per paper: `Paper [ID]` | Full Paper Title | IRR score(s)
- An AVERAGE row at the bottom

The Excel export includes color-coded IRR values (green >= 0.80, amber >= 0.60, red < 0.60), section header styling, and alternating row shading to match the on-screen appearance.

---

## Column Configuration

### Column Picker
After uploading, use the column picker to select which coding columns to include in the IRR calculation. Columns from the detected sheet version are pre-selected. Use **Select all** / **Clear** buttons for bulk actions.

### Data Types
Click the type badge (NOM / ORD / INT) next to any selected column to change its data type. This affects how Krippendorff's Alpha measures disagreement between values.

### Coder Filter
Check **Filter to specific coders only** to restrict calculations to a subset of reviewers. All reviewers found in the uploaded file are listed and checked by default.

---

## Technical Notes

- **No installation required.** The application is a single HTML file with one external dependency (SheetJS, loaded from a CDN for Excel support).
- **All computation is client-side.** No data is sent to any server.
- **Artifact filtering:** Rows where the Research Question ID is blank, a bare integer, or exactly `"Numerical (index identifier)"` are automatically excluded.
- **Missing data:** Empty cells are excluded from pairwise calculations. Metric columns with no data from any reviewer are excluded from pre-selection.
- **Methodology tooltip:** Hover the info icon (i) next to "IRR Results" to see the calculation methodology, validation notes, and aggregation logic.

---

## Project Context

This tool was developed for the **GenAI Evidence Hub**, a systematic literature review project at Learning Data Insights tracking generative AI use in educational assessment. The three evidence domains are Automated Scoring, Item Generation, and Formative Feedback.

For questions, contact the LDI project team.
