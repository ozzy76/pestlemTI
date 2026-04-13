# Threat Intelligence PESTLEM Analysis

Interactive threat intelligence analysis notebook using the PESTLEM framework, NATO Admiralty credibility ratings, and Google Gemini AI.

## Overview

This Google Colab notebook automates the collection, rating, and analysis of threat intelligence by:

1. **Collecting** intelligence from 22 curated RSS feeds across cyber, geopolitical, healthcare, and general news categories
2. **Rating** sources and information using the NATO Admiralty System (AJP-2.1)
3. **Corroborating** claims by counting independent publisher coverage
4. **Analyzing** findings through the PESTLEM framework using Google Gemini AI
5. **Reporting** via downloadable HTML and Markdown reports with executive summaries

## Quick Start

1. Open `threat_intelligence_analysis.ipynb` in Google Colab
2. **Run Cell 1** — install dependencies and initialize environment
3. **Run Cell 2** — load configuration and core functions
4. **Run Cell 3** — launch the interactive chat interface
5. Type your query and click **Analyze**

## PESTLEM Framework

Each RSS feed is tagged with one or more PESTLEM dimensions:

| Dimension | Description |
|-----------|-------------|
| **P**olitical | Government actions, policy, elections, governance |
| **E**conomic | Markets, sanctions, trade, financial crime |
| **S**ocial | Public health, demographics, civil unrest |
| **T**echnical | Cybersecurity, vulnerabilities, threat actors |
| **L**egal | Legislation, enforcement, litigation |
| **E**nvironmental | Climate, infrastructure resilience |
| **M**ilitary | Conflict, defense, geopolitical posture |

## NATO Admiralty System (AJP-2.1)

Every article is rated on two axes:

**Source Reliability (A–F)**

| Grade | Meaning | Feeds |
|-------|---------|-------|
| A | Completely Reliable | FBI Stories |
| B | Usually Reliable | Krebs, DFIR Report, Microsoft Security, Risky Business, Cyberscoop, BBC, NPR, Financial Times, Bellingcat, USAFacts, RAND, Healthcare Dive, USA Congress, News 24/680 |
| C | Fairly Reliable | Security Affairs, CSO Online, RFI International, Fierce Healthcare, The Verge, New Scientist, The Guardian, Databreaches.net, SCMP |
| D | Not Usually Reliable | Geopolitical Economy |

**Information Credibility (1–6)**

| Grade | Meaning |
|-------|---------|
| 1 | Confirmed by other sources |
| 2 | Probably true |
| 3 | Possibly true |
| 4 | Doubtful |
| 5 | Improbable |
| 6 | Cannot be judged |

Articles rated B2 or higher are flagged **ESCALATE** in the report.

## RSS Feed Sources (22 Feeds)

### Cyber / Technical
- Krebs on Security, Security Affairs, The DFIR Report, Microsoft Security, Risky Business, CSO Online, Cyberscoop, Databreaches.net, New Scientist, The Verge

### Government / Political
- FBI Stories, USAFacts, USA Congress (GovInfo)

### Geopolitical
- Bellingcat, RAND Research, Geopolitical Economy, South China Morning Post, RFI International

### Healthcare / Social
- Fierce Healthcare, Healthcare Dive, BBC Health

### General News
- BBC News, NPR Top Stories, Financial Times, News 24/680, The Guardian

## Configuration

Edit the top of Cell 2 to customize behavior:

```python
ANALYST_NAME  = "TI Agent"           # Name displayed in reports
RAG_FOLDER    = "/content/drive/MyDrive/Area/threat_intel"  # Google Drive path for RAG docs
LOOKBACK_DAYS = 5                     # How many days back to fetch articles
```

### Adding RSS Feeds

Add entries to `RSS_FEEDS` in Cell 2:

```python
{"name": "Your Feed",
 "url": "https://example.com/feed",
 "category": "cyber",
 "admiralty_source": "B",
 "pestlem": ["Technical"]}
```

## Report Output

Each analysis generates:

- **HTML report** — formatted with per-article Admiralty badges, ESCALATE banners, and a full legend
- **Markdown report** — plain-text version for archival or further processing

Both files are automatically downloaded at the end of analysis (Colab) or saved locally.

## Requirements

- Google Colab (recommended) or Jupyter with `ipywidgets`
- Google Drive (optional, for RAG document ingestion)
- Google Gemini AI access via `google.colab.ai`

### Python Dependencies (auto-installed in Cell 1)

`feedparser`, `requests`, `beautifulsoup4`, `tqdm`, `python-dateutil`, `lxml`, `markdown`, `pdfplumber`

## Notebook Structure

| Cell | Purpose |
|------|---------|
| Cell 1 | Install dependencies, detect Colab/Drive environment |
| Cell 2 | Analyst config, RSS feeds, Admiralty functions, analysis pipeline |
| Cell 3 | Interactive chat UI — submit queries, stream results, download reports |
| Cell 4 | Additional export options (re-download last report) |

## Security Considerations

- No API keys are stored in the notebook; Gemini access uses the Colab session
- RAG documents are read from Google Drive and never transmitted externally
- All analysis remains within the Colab runtime

## Limitations

- Requires an active Google Colab session with Gemini AI enabled
- English-language sources only
- Analysis is bounded by `LOOKBACK_DAYS`; older articles are filtered out
- Automated Admiralty ratings require analyst validation before dissemination

---

**UNCLASSIFIED** — For analyst use only.
