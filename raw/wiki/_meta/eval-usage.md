Type: #type/meta
Area: #area/spatial-intelligence
Date created: [[2026-05-09]]

---

# What the eval questions are for

Short version: the questions in `eval.md` are a measuring stick, not a to-do list. You don't answer them manually. Claude answers them, on demand, and rates its own answers. The ratings tell you whether the vault is getting better.

## The loop, concretely

1. **Ingest stuff into the vault** — drop papers, articles, transcripts into `inbox/`. Claude processes them into `sources/` and `wiki/`.

2. **Every few months, or whenever you're curious, run `/foundry-eval`.** Claude:
   - Takes each of the questions in `eval.md`
   - Tries to answer each one using only wiki pages (not raw sources directly)
   - Rates its own answer: `strong` / `thin` / `missing`
   - Writes the answers and ratings into the "Latest run" section of `eval.md`
   - Feeds the `thin` and `missing` findings into `index.md` as Open Questions and Candidates

3. **Read the answers.** Two things happen:
   - **Trend tells you if the wiki is improving.** If Q1 was `missing` six months ago and `thin` now and `strong` next quarter, the vault is maturing. If everything stays `missing` for a year, ingest has drifted away from the domain.
   - **Thin/missing findings tell you what to ingest next.** If Q4 (place cells / grid cells) is still `missing` after a year, that's a clear signal: go find an O'Keefe or Moser source and put it in the inbox.

That's it. It's a feedback loop for you, not homework.

## Why it's structured this way

Without an eval, wikis rot in a predictable way: you ingest whatever crosses your desk, the vault gets bigger but not more useful, and you can't tell. The lint stats (page count, orphans, keyword drift) are quantitative and don't catch *quality* drift. Eval questions are the qualitative counterweight.

The expected-rating line on each question ("Expected early rating: missing") is there so the *first* run has a baseline. If the first eval comes back better than expected, great. If it comes back worse, something's wrong with how the MANIFEST is being applied.

## A concrete example

Say in three months you've ingested 15 more sources. You run `/foundry-eval`. It comes back:

- Q1 (definition): `thin` — have Thurstone and O'Keefe, missing computational framing
- Q3 (cognitive map): `strong` — Tolman + O'Keefe + 2 critiques, well sourced
- Q4 (place/grid cells): `thin` — have the Nobel summary, missing circuit-level dissociation sources
- Q11 (LLM spatial failures): `missing` — nobody's ingested a benchmark paper yet

That output *is* your research agenda for the next quarter. Find a computational-AI definition source for Q1. Find a dissociation paper for Q4. Find a spatial-reasoning benchmark for Q11.

## When to actually run it

**Not yet.** With no sources ingested, running eval would produce entirely `missing` ratings you already expect. First useful run is after maybe 8-12 sources, probably 2-3 months in.

## How to add or change questions

- **Don't reorder or renumber existing questions** — that breaks comparability across runs.
- **Add new questions at the bottom** with a new number.
- **Retiring a question**: strike it through but leave it in place, so old-run comparisons still make sense.
- **Expected early ratings**: only set these when you first add a question, as a baseline. Don't update them to match reality — the whole point is to measure drift against the original expectation.
