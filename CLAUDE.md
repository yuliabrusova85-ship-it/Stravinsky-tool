# Stravinsky Reception History Tool — Dev Guide

## Project structure

```
index.html    — entire application (single file, ~1400 lines)
README.md     — user-facing documentation
CLAUDE.md     — this file
```

Everything — HTML, CSS, and JavaScript — lives in `index.html`. There is no build step, bundler, or package manager.

## Architecture

### Data model

Each review is a plain JS object stored in `localStorage` under the key `stravinsky_reviews`:

```js
{
  id: Number,            // Date.now() + random offset
  text: String,          // full review text
  year: Number,
  decade: Number,        // Math.floor(year / 10) * 10
  publication: String,
  author: String,
  work_reviewed: String,
  review_type: String,   // 'concert' | 'recording' | 'essay' | 'obituary' | 'book' | 'interview'
  notes: String,
  word_count: Number,
  polarity: Number,      // -1 to 1
  subjectivity: Number,  // 0 to 1
  vocab: {               // per-cluster word hit counts
    hostility: Number,
    ambivalence: Number,
    reassessment: Number,
    acceptance: Number,
    craft_influence: Number
  },
  date_added: String     // ISO 8601
}
```

### Sentiment pipeline

`analyzeSentiment(text)` → `{ polarity, subjectivity }`

Three-layer lexicon approach:
1. **Negation** — scans back 3 words for negators; flips the hit's polarity
2. **Intensifiers** — if the preceding word is an intensifier, multiplies the score by 1.5
3. **Stemming** — `stem(w)` strips common suffixes before checking `POSITIVE_WORDS` / `NEGATIVE_WORDS`

`scoreVocabulary(text)` → per-cluster hit counts using `VOCABULARY` object.

Both functions are called on every `addReview()` and `saveEdit()` call. When the vocabulary clusters are edited and saved via `saveVocabulary()`, all existing reviews are re-scored.

### Tab system

`switchTab(name, btn)` — activates the named tab content div and calls the appropriate render function:

| Tab | Render function |
|-----|----------------|
| browse | `renderTable()` |
| analysis | `renderAnalysis()` |
| visualize | `renderCharts()` (delayed 100ms for DOM) |
| vocabulary | `renderVocabulary()` |
| report | user-triggered via button |

### Charts

Four Chart.js instances stored in `charts = {}`. All destroyed and re-created on each `renderCharts()` call to avoid canvas conflicts.

### External libraries (CDN)

| Library | Version | Use |
|---------|---------|-----|
| Chart.js | 4.4.0 | All visualizations |
| PDF.js | 3.11.174 | Client-side PDF text extraction |
| mammoth.js | 1.6.0 | Client-side DOCX text extraction |

PDF.js requires setting `pdfjsLib.GlobalWorkerOptions.workerSrc` before use — this is done at the bottom of the script block.

## Key functions

| Function | Purpose |
|----------|---------|
| `addReview()` | Validates form, runs duplicate check, scores sentiment, saves |
| `saveEdit()` | Updates existing review in-place, re-runs sentiment |
| `openEdit(id)` | Populates and opens the edit modal |
| `openCite(id)` | Generates Chicago/MLA/APA citations and opens the cite modal |
| `buildCitations(r)` | Returns `{ chicago, mla, apa }` strings |
| `extractPDF(file)` | Async — uses PDF.js to extract all page text |
| `extractDOCX(file)` | Async — uses mammoth.js to extract raw text |
| `handleDocImport(event)` | Orchestrates PDF/DOCX import, pre-fills form |
| `importCSV(event)` | Bulk imports from CSV file |
| `exportAnalysisCSV()` | Downloads analysis results as CSV |
| `generateReport()` | Builds and displays the text report |
| `saveVocabulary()` | Persists edited vocabulary and re-scores all reviews |
| `renderTable()` | Filters/sorts/renders the Browse tab table |
| `renderAnalysis()` | Renders stats cards and analysis table |
| `renderCharts()` | Destroys and re-renders all four Chart.js charts |
| `renderVocabulary()` | Renders editable vocabulary cluster cards |
| `saveData()` | `localStorage.setItem(...)` wrapper |
| `showToast(msg, type)` | Shows bottom-right notification |

## CSS variables

All colors use CSS custom properties defined in `:root`:

```css
--primary: #2c3e50
--accent: #3498db
--accent-dark: #2980b9
--success: #27ae60
--warning: #f39c12
--danger: #e74c3c
--bg: #f5f6fa
--card: #ffffff
--text: #2c3e50
--text-light: #7f8c8d
--border: #dcdde1
```

## Common tasks

**Add a new work to the dropdown**
Find the `<select id="reviewWork">` element and add an `<option>` following the existing pattern.

**Add a new vocabulary cluster**
1. Add a new key to the `VOCABULARY` object with an array of terms
2. Add it to `POSITIVE_WORDS` or `NEGATIVE_WORDS` as appropriate
3. Add `cluster-yourname` CSS rule in the vocabulary cluster section
4. Add it to the `clusters` array in `generateReport()` and `renderCharts()` cluster chart

**Extend the sentiment lexicon**
Add words directly to `POSITIVE_WORDS` or `NEGATIVE_WORDS` Sets. Words are checked against both the raw form and the stemmed form, so adding `beautiful` also catches `beautifully`.

**Change polarity thresholds**
Edit `getSentimentLabel()` — current thresholds: `< -0.15` Hostile, `< 0.1` Ambivalent, `< 0.35` Receptive, `≥ 0.35` Enthusiastic.
