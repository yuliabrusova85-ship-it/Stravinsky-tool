# Stravinsky Reception History Tool

**Live:** https://yuliabrusova85-ship-it.github.io/Stravinsky-tool/

A single-file browser app for computational analysis of critical reception of Igor Stravinsky's serial-period works (1952–1966). Built for musicological research — no server, no install, runs entirely in the browser.

## What it does

| Tab | Function |
|-----|----------|
| **Add Reviews** | Enter reviews manually, extract text from PDF/DOCX, or bulk import via CSV |
| **Browse Data** | Search and filter the review database by keyword, decade, or type |
| **Analysis** | Sentiment scores and vocabulary cluster breakdown per review |
| **Visualize** | Four Chart.js charts: polarity timeline, cluster trends by decade, sentiment by publication, review volume |
| **Report** | Auto-generated research summary downloadable as `.txt` |
| **Vocabulary** | Edit the critical lexicon clusters and re-score all reviews instantly |

## Sentiment Engine

Uses a **lexicon-based NLP approach** with three layers:

- **Negation detection** — looks back up to 3 words for negators (`not`, `never`, `hardly`, `lacks`, `fails`, etc.) and flips polarity accordingly. "Not beautiful" scores negative.
- **Intensifier boosting** — words preceded by `very`, `deeply`, `remarkably`, `profoundly`, etc. score 1.5×.
- **Stemming** — strips common suffixes (`-ly`, `-ing`, `-ed`, `-ness`, `-ful`, `-tion`) so `cerebrally`, `cerebral`, and `cerebrally` all match the same root.

Five vocabulary clusters track critical discourse trends:

| Cluster | Purpose |
|---------|---------|
| **Hostility** | Negative reaction language: *incomprehensible, arid, sterile, mechanical* |
| **Ambivalence** | Guarded response: *challenging, austere, concentrated, severe* |
| **Reassessment** | Positive reappraisal: *masterpiece, visionary, seminal, groundbreaking* |
| **Acceptance** | Full embrace: *luminous, sublime, transcendent, poignant* |
| **Craft/Influence** | Technical discourse: *serial, twelve-tone, dodecaphonic, hexachord* |

## Features

- **PDF/DOCX import** — drag a review document onto the import zone; text is extracted client-side (PDF.js + mammoth.js) and pre-filled in the form
- **Citation formatter** — generates Chicago, MLA, and APA citations from stored metadata with one-click copy
- **Duplicate detection** — warns before adding a review that matches an existing author + work within ±1 year
- **Export** — analysis results as CSV, reports as TXT
- **Persistent storage** — all data saved in `localStorage`; works offline after first load

## Works covered

Stravinsky's serial period (1952–1966):

Cantata · Septet · Three Songs from Shakespeare · In Memoriam Dylan Thomas · Canticum Sacrum · Agon · Threni · Movements · Epitaphium · Double Canon · A Sermon, a Narrative and a Prayer · The Flood · Abraham and Isaac · Variations (Aldous Huxley in Memoriam) · Introitus · Requiem Canticles · The Owl and the Pussy-Cat

## Usage

Open `index.html` in any modern browser. No build step, no dependencies to install — CDN scripts load automatically.

To get started quickly, click **Load Sample Data** on the Add Reviews tab to populate 10 example reviews spanning 1956–1985.

## CSV Import Format

```
year,publication,author,work_reviewed,review_type,text,notes
1958,Musical Times,Anonymous Critic,Threni (1958),concert,"Full review text here...","Your research notes"
```

`review_type` values: `concert` · `recording` · `essay` · `obituary` · `book` · `interview`

## Tech stack

- Vanilla HTML/CSS/JS — no framework
- [Chart.js 4.4](https://www.chartjs.org/) — charts
- [PDF.js 3.11](https://mozilla.github.io/pdf.js/) — PDF text extraction
- [mammoth.js 1.6](https://github.com/mwilliamson/mammoth.js) — DOCX text extraction
