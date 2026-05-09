# LLM Knowledge Base — Operating Instructions

This Obsidian vault is an LLM-managed knowledge base following the Karpathy pattern.
The LLM writes and maintains all wiki content. The human rarely edits directly.

## Directory Structure

```
raw/            — Source documents ingested into the KB
  articles/     — Web articles (clipped via Obsidian Web Clipper)
  papers/       — Academic papers and technical reports
  repos/        — Notes/READMEs from code repositories
  datasets/     — Dataset descriptions and samples
  images/       — Downloaded images referenced by raw docs
wiki/           — LLM-compiled wiki (the core knowledge base)
  concepts/     — Concept articles (one per topic)
  sources/      — Source summaries (one per raw document)
  indexes/      — Index files, category pages, and cross-references
output/         — LLM-generated outputs from queries
  slides/       — Marp-format slide decks
  visualizations/ — matplotlib/chart images
  reports/      — Research reports and Q&A results
tools/          — CLI tools for operating on the KB
```

## Core Workflows

### 1. Ingest (raw → source summary)
When new files appear in `raw/`, create a corresponding source summary in `wiki/sources/`:
- Read the raw document fully
- Discuss key takeaways with the user before writing — don't silently process
- Write a structured summary with metadata (title, date, authors, URL, tags)
- Extract key concepts and link to existing concept articles using `[[wikilinks]]`
- Note any new concepts that need articles
- Append an entry to `wiki/log.md` (see Logging below)

### 2. Compile (source summaries → concept articles)
After ingesting, update the wiki:
- Create new concept articles in `wiki/concepts/` for newly discovered topics
- Update existing concept articles with new information and source backlinks
- Update index files in `wiki/indexes/`
- Maintain `wiki/indexes/Master Index.md` as the central directory

### 3. Q&A (query → output)
When answering questions against the KB:
- First consult `wiki/indexes/Master Index.md` to find relevant articles
- Read relevant concept articles and source summaries
- Generate output as markdown files in `output/` when appropriate
- **File good answers back into the wiki as new pages** — comparisons, analyses, and connections discovered through Q&A are valuable and should compound in the KB, not disappear into chat history
- Append an entry to `wiki/log.md`

### 4. Lint / Health Check
Periodically run health checks:
- Find broken wikilinks
- Find orphaned articles (no backlinks)
- Find inconsistent data across articles
- Find stale claims that newer sources have superseded
- Suggest new concept articles for frequently mentioned but unlinked topics
- Suggest missing data that could be filled via web search
- Suggest new questions to investigate and new sources to look for
- Append an entry to `wiki/log.md`

## Logging

`wiki/log.md` is a chronological, append-only record of what happened and when — ingests, queries, lint passes. Each entry uses the format:

```
## [YYYY-MM-DD] action | Title
```

For example: `## [2026-04-05] ingest | Attention Is All You Need`

This gives a timeline of the wiki's evolution and helps the LLM understand what's been done recently. The log is parseable with simple unix tools (e.g. `grep "^## \[" wiki/log.md | tail -5`).

## Research Topics

Multiple research topics coexist in a single flat wiki. Topics are distinguished by **tags**, not subdirectories. This preserves cross-referencing — a source relevant to multiple topics naturally gets tagged with all of them, and connections between topics can surface organically.

- Every concept and source gets one or more topic tags in frontmatter (e.g. `#transformer`, `#climate-policy`)
- Create a **topic index page** in `wiki/indexes/` for each research area (e.g. `wiki/indexes/Transformers.md`) that collects all relevant concepts and sources for that topic
- The Master Index links to all topic index pages
- When ingesting, ask the user which topic(s) a source belongs to if it's not obvious
- During lint, check for sources/concepts that may be missing topic tags

## Conventions

- All wiki content uses Obsidian-flavored markdown with `[[wikilinks]]`
- Every concept article has YAML frontmatter: title, aliases, tags, created, updated
- Every source summary has YAML frontmatter: title, source_type, url, authors, date, tags
- Tags use lowercase-kebab-case: `#machine-learning`, `#transformer`
- Images are stored in `raw/images/` and referenced with relative paths
- Index files are maintained automatically — never edit manually
- Use Marp front matter (`marp: true`) for slide decks in `output/slides/`

## Commands for the LLM

When the user says:
- **"ingest"** or **"compile"** — Process all new raw/ files into the wiki
- **"lint"** or **"health check"** — Run integrity checks on the wiki
- **"search [query]"** — Search across the wiki and return results
- **"report on [topic]"** — Generate a research report in output/
- **"slides on [topic]"** — Generate a Marp slide deck in output/slides/
- **"status"** — Show KB statistics (article count, word count, recent changes)
