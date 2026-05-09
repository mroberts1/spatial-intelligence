---
title: Spatial Intelligence Wiki Log
type: meta
created: 2026-05-08
updated: 2026-05-08
---

# Spatial Intelligence Wiki Log

Append-only chronological record of all vault operations.

**Format:** `## [YYYY-MM-DD] operation | subject`

**Operations:** ingest, compile, update, query, lint, eval, prune, create, archive

When this file exceeds 500 entries, rotate: rename to log-YYYY.md, start fresh.

---

## [2026-05-08] create | Foundry v2 vault initialized

**Domain:** AI Spatial Intelligence — World models, neural graphics, embodied AI, 3D representations

**Structure:**
- `raw/` — Source documents (articles, papers, notes)
- `wiki/concepts/` — Concept articles (one per topic)
- `wiki/sources/` — Source summaries (one per raw document)
- `wiki/indexes/` — Master Index, topic-specific indexes
- `wiki/_meta/` — Evaluation framework, index, log
- `output/` — Generated reports, slide decks

**Foundation documents:**
- CLAUDE.md — Operating instructions (Karpathy pattern)
- Master Index (wiki/indexes/)
- Evaluation questions (wiki/_meta/eval.md)

---

## [2026-05-08] ingest | 3D as Code + From Words to Worlds

**Source articles:**
- [[3D as Code]] (World Labs)
- [[From Words to Worlds]] (Fei-Fei Li)

**Concepts created:**
- [[Spatial Intelligence]]
- [[World Models]]
- [[3D Representations]]
- [[Neural Graphics]]
- [[Embodied AI]]

**Key synthesis:**
- 3D as spatial code (analogy: code ↔ 3D, language ↔ neural graphics, chips ↔ simulation)
- World models as three-pillar architecture (generative, multimodal, interactive)
- Spatial intelligence as AI's frontier beyond language

---

## [2026-05-08] ingest | Streaming 3DGS Worlds on the Web

**Source article:**
- [[Streaming 3DGS Worlds on the Web]] (World Labs / Spark team)

**Concepts created/updated:**
- [[3D Gaussian Splatting]] (new)
- [[Neural Graphics]] (connected)

**Key synthesis:**
- 3DGS as practical neural graphics implementation
- Level-of-Detail splat trees for scalability (O(budget) traversal)
- Progressive streaming (.RAD format) and virtual memory for infinite worlds
- 40M+ splat scenes on mobile/VR/desktop via Spark 2.0

**Architectural insight:**
- Full stack now visible: 3D Representations → 3DGS (neural graphics) → World Models (generation) → Spark (rendering)
- Marble generates 3DGS worlds; Spark streams and renders them at scale

---

## [2026-05-08] create | Evaluation framework

**Files created:**
- `wiki/_meta/eval.md` — 13 evaluation questions probing vault maturity
- `wiki/_meta/eval-usage.md` — Instructions for evaluation loop
- `wiki/_meta/index.md` — Content catalog, open questions, candidates
- `wiki/_meta/log.md` — This file

**Evaluation status:** Framework ready; first meaningful run after ~8-12 sources (2-3 months in)

**Open questions identified:**
- Multimodal integration in world models
- LoD system scaling and limits
- Spatial cognition connection (AI vs. biology)
- Benchmarks for spatial reasoning in VLMs
- Physics + learning tradeoffs in hybrid systems

**Next ingest priorities:**
1. Multimodal world model architecture (Marble paper)
2. Sim-to-real robotics
3. Spatial reasoning benchmarks
4. Physics integration in neural systems

---

## Statistics & Progress

| Metric | Current | Target |
|--------|---------|--------|
| Sources ingested | 3 | 8-12 (before first eval) |
| Concepts | 6 | Growing |
| Candidates | 8 | Track per eval |
| Evaluation status | Framework ready | First run at 8-12 sources |

**Vault health:** Initializing. Spatial stack architecture visible after 3 sources. Multimodal learning and applications still underdeveloped. Ready to scale ingest.

---

## [2026-05-09] update | CLAUDE.md rewritten to v2 governance

Replaced MacBook-side v1 CLAUDE.md with full v2 governance document:
- Voice anchor rewritten to accurately reflect vault domain (AI spatial intelligence — world models, neural graphics, 3DGS, embodied AI)
- Scope section added: AI/computational focus is primary; cognitive science/neuroscience in scope only as grounding or contrast
- Sibling vaults updated: ~/generative-art as primary sibling; ~/interasia-pop and ~/tiwchh not directly linked
- Tag taxonomy replaced with 10 domain-accurate tags matching existing content
- Directory conventions aligned to MacBook-established structure (wiki/concepts/, wiki/sources/, Title Case)
- Changes from v1 table documents what was superseded

## [2026-05-09] update | v2 front-matter patch on 4 source notes

Added missing v2 fields to all source notes in wiki/sources/: `tier:`, `last_verified:`, `superseded_by:`. All four assigned `tier: primary` (World Labs official blog posts and technical articles). No content changes.
