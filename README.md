# 📊 outbound-final-report

A Claude skill that generates professional B2B outbound reports in `.docx` format — available in two modes: **monthly** (for active clients) and **closing** (for finished projects).

---

## What this skill does

- Reads the performance CSV exported from Snov.io (or Instantly, Apollo, Lemlist)
- Reads a screenshot of the full lead base to map the ICP built throughout the project
- Reads client context from a website, presentation, or uploaded file
- Calculates KPIs: delivery rate, open rate, reply rate, bounce rate, and notable engagements
- **Monthly mode:** runs a web search for up-to-date market data and generates actionable insights backed by validated sources
- **Closing mode:** generates project learnings and continuity recommendations
- Delivers a formatted `.docx` with cover page, tables, visual KPI blocks, and consistent design

---

## Required inputs

| Input | Format | Required |
|---|---|---|
| Report mode | monthly or closing | Yes |
| Client name and point of contact | text | Yes |
| Period covered | month/year or date range | Yes |
| Performance CSV | `.csv` exported from the outbound tool | Yes |
| Screenshot of the full lead base | image | Yes |
| Client context | website URL, presentation URL, or `.pptx`/`.pdf` file | Yes |
| Agency name (report author) | text | Yes |
| Previous months data (accumulated) | text or spreadsheet | Optional |

---

## Supported tools

- **Snov.io** (default — PT and EN)
- Instantly
- Apollo
- Lemlist

---

## How to install

1. Download the `SKILL.md` file from this repository
2. In Claude, go to **Settings > Skills**
3. Upload the file
4. Done — Claude will automatically recognize when you need an outbound report

---

## How to use

Just start a conversation with something like:

> "Generate the monthly outbound report for client X"

> "Let's close the final report for project Y"

Claude will request the necessary inputs and deliver the `.docx` at the end.

---

## Report structure

### Monthly mode
`Cover` → `Project context` → `Lead base & ICP` → `Month performance` → `Notable engagements` → `Insights & improvement points` → `Next steps`

### Closing mode
`Cover` → `Project context` → `Lists & ICP` → `Overall performance` → `Notable engagements` → `Learnings` → `Project deliverables` → `Final remarks`

---

## Built by

[AN1](https://an1.com.br) — B2B Marketing & Sales
