# The core idea
Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation. Ask a subtle question that requires synthesizing five documents, and the LLM has to find and piece together the relevant fragments every time. Nothing is built up. NotebookLM, ChatGPT file uploads, and most RAG systems work this way.

The idea here is different. Instead of just retrieving from raw documents at query time, the LLM incrementally builds and maintains a persistent wiki — a structured, interlinked collection of markdown files that sits between you and the raw sources. When you add a new source, the LLM doesn't just index it for later retrieval. It reads it, extracts the key information, and integrates it into the existing wiki — updating entity pages, revising topic summaries, noting where new data contradicts old claims, strengthening or challenging the evolving synthesis. The knowledge is compiled once and then kept current, not re-derived on every query.

This is the key difference: the wiki is a persistent, compounding artifact. The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read. The wiki keeps getting richer with every source you add and every question you ask.

You never (or rarely) write the wiki yourself — the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions. The LLM does all the grunt work — the summarizing, cross-referencing, filing, and bookkeeping that makes a knowledge base actually useful over time. In practice, I have the LLM agent open on one side and Obsidian open on the other. The LLM makes edits based on our conversation, and I browse the results in real time — following links, checking the graph view, reading the updated pages. Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

This can apply to a lot of different contexts. A few examples:

Personal: tracking your own goals, health, psychology, self-improvement — filing journal entries, articles, podcast notes, and building up a structured picture of yourself over time.
Research: going deep on a topic over weeks or months — reading papers, articles, reports, and incrementally building a comprehensive wiki with an evolving thesis.
Reading a book: filing each chapter as you go, building out pages for characters, themes, plot threads, and how they connect. By the end you have a rich companion wiki. Think of fan wikis like Tolkien Gateway — thousands of interlinked pages covering characters, places, events, languages, built by a community of volunteers over years. You could build something like that personally as you read, with the LLM doing all the cross-referencing and maintenance.
Business/team: an internal wiki maintained by LLMs, fed by Slack threads, meeting transcripts, project documents, customer calls. Possibly with humans in the loop reviewing updates. The wiki stays current because the LLM does the maintenance that no one on the team wants to do.
Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives — anything where you're accumulating knowledge over time and want it organized rather than scattered.

# Architecture
There are three layers:

Raw sources — your curated collection of source documents. Articles, papers, images, data files. These are immutable — the LLM reads from them but never modifies them. This is your source of truth.

The wiki — a directory of LLM-generated markdown files. Summaries, entity pages, concept pages, comparisons, an overview, a synthesis. The LLM owns this layer entirely. It creates pages, updates them when new sources arrive, maintains cross-references, and keeps everything consistent. You read it; the LLM writes it. Everything in the wiki is treated as canonical — the current best understanding given all sources processed so far. When information is clearly missing, the LLM marks the gap explicitly with `**Missing Info**: <what's missing>` rather than filling it with plausible-sounding content. Use-case agents may define page templates — standardized structures with frontmatter, required sections, and Missing Info defaults for each page type — to keep the wiki consistent as it grows.

The schema — a document (e.g. CLAUDE.md for Claude Code or AGENTS.md for Codex) that tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow when ingesting sources, answering questions, or maintaining the wiki. This is the key configuration file — it's what makes the LLM a disciplined wiki maintainer rather than a generic chatbot. The schema is the most leveraged file in the system — a good convention improves every future operation, and a bad one left uncorrected degrades the wiki silently. You and the LLM co-evolve the schema over time as you figure out what works for your domain; when a workflow doesn't fit, the LLM proposes an edit and applies it on agreement.

# Feedback loops
The pattern's value comes from three reinforcing loops. Naming them helps you notice when they're healthy and when they've stalled.

**The compounding loop.** Ingest → richer wiki → better queries → valuable answers filed back → richer wiki. This is the core engine. Each source doesn't just add to the wiki — it thickens the web of cross-references that makes the next source's integration richer and the next query's synthesis deeper. The wiki compounds because every new piece of knowledge connects to everything already there.

**The persona-refinement loop.** Bootstrap → persona → ingestion calibrated to persona → user's responses refine the persona → better calibration. Early sessions produce generic summaries because the LLM doesn't yet know what the user cares about. Over time, as the persona sharpens, the LLM's emphasis, analogies, and deprioritization choices become increasingly tailored. The persona section of the schema should visibly change over the life of the wiki.

**The gap-driven acquisition loop.** Missing Info markers → user notices gaps → seeks new sources → ingest → fills gaps → reveals new gaps. The wiki doesn't just organize what you know — it tells you what you don't know. Missing Info markers are a reading list in disguise: they drive the user toward the sources that would be most valuable to add next.

# Core operations
Three operations form the always-on backbone of the pattern. These are the minimum viable wiki.

- **Ingest**. You drop a new source into the raw collection and tell the LLM to process it. An example flow: the LLM reads the source, discusses key takeaways with you, writes a summary page in the wiki, updates the index, updates relevant entity and concept pages across the wiki, and appends an entry to the log. A single source might touch 10-15 wiki pages. Personally I prefer to ingest sources one at a time and stay involved — I read the summaries, check the updates, and guide the LLM on what to emphasize. But you could also batch-ingest many sources at once with less supervision. It's up to you to develop the workflow that fits your style and document it in the schema for future sessions.

- **Query**. You ask questions against the wiki. The LLM searches for relevant pages, reads them, and synthesizes an answer with citations. Answers can take different forms depending on the question — a markdown page, a comparison table, a slide deck (Marp), a chart (matplotlib), a Mermaid diagram, a canvas. The important insight: good answers can be filed back into the wiki as new pages. A comparison you asked for, an analysis, a connection you discovered — these are valuable and shouldn't disappear into chat history. This way your explorations compound in the knowledge base just like ingested sources do.

- **Lint**. The primary job of linting is fighting entropy. Wikis decay: pages go stale, cross-references break, new sources silently contradict old claims, and the knowledge base drifts from its sources. Humans abandon wikis because this entropy accumulates faster than anyone wants to deal with. Linting is the countermeasure. On request, the LLM sweeps the wiki for: contradictions between pages, stale claims that newer sources have superseded, orphan pages with no inbound links, important concepts mentioned but lacking their own page, missing cross-references, and data gaps that could be filled with a web search or a new source. The LLM is also good at suggesting new questions to investigate and new sources to look for — lint passes don't just repair the wiki, they drive the gap-driven acquisition loop forward.

# Visual structure
Mermaid diagrams are strongly encouraged in synthesized pages — topic pages, comparisons, saved query results. Place the diagram high in the page, before the prose, as a visual entry point. Think of each diagram as a mini knowledge graph: a structured web of concepts joined by labeled, semantic relationships. Edges matter more than nodes — every arrow should carry a short verb phrase that names the relationship ("enables", "contradicts", "depends on", "is measured by"), so that any two nodes and their connecting edge read as a meaningful sentence. Keep diagrams tight (5–10 nodes); if everything is on the diagram, nothing is. If a diagram would be vacuous (fewer than 3 meaningful nodes), skip it rather than forcing one.

# Optional operations
These are not required for the pattern to work, but each adds a distinct capability. Use-case agents can adopt any combination.

- **Bootstrap**. Before the first ingest, the LLM interviews the user to establish a persona — who they are, what they care about, and how they think. Three questions, asked as a batch: (1) what's your background and role? (2) what are you trying to accomplish with this wiki — what's the goal or driving question? (3) what mental models or frameworks do you already use to think about this domain? The answers are stored in the schema file and inform every downstream operation — what to emphasize in summaries, what analogies to reach for, what to deprioritize. Without bootstrapping, the persona-refinement loop still works — it just starts cold, and the first few ingests produce less calibrated output.

- **Adaptive ingestion questioning**. During ingestion, the LLM may ask up to three questions to understand the source in context — why it was added, how it connects to the bigger picture, what the user wants to get out of it. The frequency is adaptive: early ingests, when the persona is thin and the wiki is sparse, trigger more questions. As the wiki grows and the LLM develops a model of the user's interests and goals, it asks less — the existing wiki provides enough context to infer the answers. The first five sources might get three questions each; source fifty might get none. This feeds the persona-refinement loop directly.

- **Quiz**. The LLM tests the user's understanding of wiki content through a Feynman-style teaching session: it picks a concept, teaches it simply using analogies grounded in the user's persona, then poses a conceptual question — not recall, but application ("why would X fail if you did Y?"). Based on the answer, it either advances to deeper nuance or clarifies the gap before moving on. The goal is intuition, not coverage: the user should walk away able to explain the concept in their own words. Gaps discovered during quizzing can be filed back into the wiki as open questions, closing the loop between learning and knowledge maintenance.

# Indexing and logging
Two special files help the LLM (and you) navigate the wiki as it grows. They serve different purposes:

- `index.md` is content-oriented. It's a catalog of everything in the wiki — each page listed with a link, a one-line summary, and optionally metadata like date or source count. Organized by category (entities, concepts, sources, etc.). The LLM updates it on every ingest. When answering a query, the LLM reads the index first to find relevant pages, then drills into them. This works surprisingly well at moderate scale (~100 sources, ~hundreds of pages) and avoids the need for embedding-based RAG infrastructure.

- `log.md` is chronological. It's an append-only record of what happened and when — ingests, queries, lint passes. A useful tip: if each entry starts with a consistent prefix (e.g. ## [2026-04-02] ingest | Article Title), the log becomes parseable with simple unix tools — grep "^## \[" log.md | tail -5 gives you the last 5 entries. The log gives you a timeline of the wiki's evolution and helps the LLM understand what's been done recently.

# Why this works
The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else.

The idea is related in spirit to Vannevar Bush's Memex (1945) — a personal, curated knowledge store with associative trails between documents. Bush's vision was closer to this than to what the web became: private, actively curated, with the connections between documents as valuable as the documents themselves. The part he couldn't solve was who does the maintenance. The LLM handles that.

# Note
This document is intentionally abstract. It describes the idea, not a specific implementation. The three-layer architecture (sources, wiki, schema), the core operations (ingest, query, lint), and the feedback loops are the load-bearing structure of the pattern. Everything else — bootstrapping, adaptive questioning, quizzing, page templates, visual conventions — is optional and modular. Pick what's useful for your domain, ignore what isn't. The exact directory structure, the schema conventions, the page formats, the tooling — all of that will depend on your domain, your preferences, and your LLM of choice. This document's only job is to communicate the pattern.

# TASK
Help me implement the specific idea described, by creating a `*.agent.md` file in this repo.

First, ask me 7 questions to identify the better strategy, clarify ambiguities, and fill the gaps to reach the goal.
Then, proceed with writing the markdown agent file.
