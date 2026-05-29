# LLM Wiki for Conferences

A worked adaptation of the [LLM Wiki pattern](LLM_WIKI_pattern.md) to one concrete use case: capturing everything you learn at a multi-day conference. Read the pattern first — it defines the three-layer architecture (sources / wiki / schema), the core operations (ingest, query, lint), and the feedback loops. This file describes only what changes when the domain is a conference, and gives you enough to stand up your own wiki and adapt it.

## Why a conference fits the pattern

Conferences are a stress test for knowledge management: high volume, time pressure, input arriving in many formats (abstracts, slides, your own scrappy notes, hallway conversations, photos of a slide, links a speaker drops), and in unpredictable bursts. You take messy notes, discover cross-talk connections, and by day 3 you've forgotten what impressed you on day 1.

The pattern fits because:
- **Input is heterogeneous and messy** — the wiki accepts whatever your LLM can read (text, photos, links) and normalizes it into consistent pages.
- **Knowledge compounds across talks** — each ingested talk thickens the cross-reference web, so the next talk integrates faster and the next query synthesizes deeper. Two talks that never reference each other get connected through a shared topic page.
- **The artifact outlives the event** — afterwards you have a searchable, interlinked knowledge base, not a folder of half-readable notes.

## Architecture: the same three layers, conference page types

The three layers are unchanged. What's conference-specific is the set of page types in the wiki layer:

```
sources/         ← raw, immutable: schedule, abstracts, your notes, fetched material
wiki/            ← LLM-maintained
  talks/         ← one page per attended talk (the core unit)
  speakers/      ← one page per speaker of interest
  topics/        ← concept pages — where cross-talk synthesis lives
  reflections/   ← end-of-day synthesis
  preparation/   ← interest analysis + recommended pre-reading
  syntheses/     ← saved query results worth keeping
<schema file>    ← CLAUDE.md / AGENTS.md: instructions + persona + conventions
index.md         ← content catalog
log.md           ← chronological audit trail
```

Two page types carry most of the conference-specific weight:
- **Talk pages** are the atomic unit — one per session, holding the abstract, your raw notes, a normalized summary, takeaways, and links out to speakers and topics.
- **Topic pages** are where the compounding happens — a topic touched by three talks becomes the place their threads get woven together.

The rest (speakers, reflections, preparation, syntheses) are conveniences you add lazily, on first use.

## The persona, inferred from what you chose to attend

Bootstrap a persona before the first ingest (see the pattern's *Bootstrap* operation). The conference twist: **you've already revealed a lot by which talks you picked.** Your schedule is itself a persona signal. If you annotate each chosen talk with *why* you chose it, the LLM can infer your background, goals, and what to de-emphasize without a long interview.

The persona is the most leveraged part of the schema: with it, summaries reach for analogies from your field, skip the sub-topics you don't care about, and tie examples back to projects you're actually working on. Without it, every summary is generic. It sharpens over time — every emphasis correction you make during ingestion feeds back into it.

## Intent capture: the richest conference signal

Knowing *why* you're attending each talk is the single most valuable input — it calibrates every summary, connection, and quiz. Capture it up front. If your schedule already has a "why" column, use it. If not, answer (or have the LLM ask you) a batch like:

1. **What are you building right now?** — Reveals which talks connect to something concrete vs. pure exploration.
2. **Which talks did you almost skip, and why keep them?** — Ambivalence maps to genuine-but-uncomfortable learning goals.
3. **What's currently painful in your work?** — Separates "fix a problem" talks from "explore a possibility" talks.
4. **Which topics do you feel you *should* understand but don't yet?** — High-priority targets for extra emphasis and pre-reading.
5. **What mental models do you already use?** — These become the analogical bridges in your summaries.
6. **Where does each talk sit on your comfort-zone spectrum?** — An "exposure" goal vs. a "mastery" goal changes how much depth to keep.
7. **Are you attending any talk for reasons other than the topic?** — Networking, a speaker's reputation, curiosity — this shapes emphasis too.

Store the answer per talk (e.g. a `why_i_am_interested` field) and let it weight normalization: if a talk covers method X and framing Y but you came for X, the summary leads with X. Revisit it during the end-of-day pass — did the talk deliver on the "why"?

## Operations: the core three, plus a conference rhythm

The core operations (ingest, query, lint) work as defined in the pattern. A conference adds time-shaped operations:

**Before — Preparation.** Analyze schedule + persona → interest clusters, recommended pre-reading, and explicit exclusions ("skipping X because…"). Output a topic map and reading stubs. Re-run after each day, since what you actually attend reveals more than what you planned.

**During — Ingest, between talks.** Drop raw notes; get a normalized page back. The LLM pastes your notes verbatim, writes a grounded summary, extracts takeaways, creates/updates the relevant topic and speaker pages, flags contradictions with earlier talks, and updates the index. A single talk can touch 5–15 pages.

**After each day — Reflective pass.** The highest-value conference operation. Re-read the day's talk pages, write a daily reflection (themes that emerged, surprises, contradictions, open questions), update topic pages with the day's cross-talk synthesis, and propose deep-dive directions and a quiz scope for tomorrow.

**Anytime — Query, Quiz, Lint.** Query and lint as in the pattern. Quiz is worth calling out: use a Feynman-style session (the pattern's *Quiz* operation) grounded in your wiki pages and calibrated to your persona — application questions, not recall.

## A loose talk-page template

Page templates keep the wiki consistent as it grows. The talk page is the one worth sketching; adapt the rest freely:

```markdown
---
type: talk
day: N
datetime: YYYY-MM-DDTHH:MM
speakers: [[speaker-slug]]
why_i_am_interested: <one line, or Missing Info>
tags: [talk]
---

# <Talk Title>

## Abstract
> <verbatim, or **Missing Info**: not provided>

## Why I'm interested
<your rationale — calibrates everything below>

## Raw notes
<!-- verbatim paste; never edited -->

## Normalized summary
<grounded in abstract + notes, weighted toward your "why">

## Key takeaways
- <bullet, cited>

## Concepts introduced
- [[topic-slug]] — <one-line context>

## Open questions
- ...

## Connections
- Related talks / topics; builds-on / contradicts

## Sources
- sources/notes/...  ·  sources/abstracts/...
```

Derive the other page types (speaker, topic, reflection, preparation) the same way: a little frontmatter, a few required sections, and `**Missing Info**: <what>` wherever there's no grounded content. Synthesized pages (topic, reflection, synthesis) read best when they open with a small Mermaid knowledge-graph diagram — see the pattern's *Visual structure* note.

## Conventions worth fixing early

Cheap to set, expensive to retrofit — write these into your schema before the first ingest:
- **Filenames:** kebab-case and predictable (`talks/dayN-<slug>.md`, `speakers/<lastname>-<firstname>.md`, `topics/<concept>.md`).
- **Links:** use your markdown viewer's wikilink syntax so the graph view works (Obsidian-style `[[slug]]` is convenient, and lets visual learners browse the graph).
- **Dates:** ISO 8601; timestamp log entries `[YYYY-MM-DD HH:MM]`.
- **Tags:** a small standard set (`#talk #speaker #topic #reflection`) plus signal tags (`#open-question`, `#contradiction`, `#missing-info`).
- **Discipline (unchanged from the pattern):** never invent — mark `**Missing Info**`; cite every claim; `sources/` is read-only; log every operation.

## Setting up your own

1. **Read [`LLM_WIKI_pattern.md`](LLM_WIKI_pattern.md)** for the architecture, operations, and feedback loops.
2. **Write a schema file** (CLAUDE.md / AGENTS.md) for your LLM: drop in your persona, the page types above, the conventions, and the operations rhythm. This is the most leveraged thing you'll write.
3. **Bootstrap** the empty `sources/` and `wiki/` scaffolding plus `index.md` and `log.md`.
4. **Load the schedule** into `sources/`, create a stub talk page per session you plan to attend, and capture your intent (the questions above).
5. **Run preparation** to get a topic map and reading list before day 1.
6. **During the conference:** ingest between talks, reflect at end of day, query and quiz whenever useful.
7. **Co-evolve the schema** as you learn what fits — when a workflow doesn't work, have the LLM propose the edit and apply it. The schema you finish with will look different from the one you started with, and that's the point.
