---
name: interactive-learning
description: Run a Feynman-style interactive teaching and quizzing session that builds durable intuition through progressive disclosure, calibrated conceptual questions, and adaptive depth. Use this skill whenever the user explicitly asks to be quizzed, drilled, taught interactively, or wants a deep, back-and-forth explanation — including phrases like "quiz me", "drill me on", "teach me X interactively", "Feynman me on X", "go deep on X", "deeply explain X", "test my understanding of X", "interactive learning on X". Do NOT trigger this skill for plain "explain X", "what is X?", "summarize X", "give me an overview of X", or "how does X work?" — those are normal explanation tasks, not interactive learning sessions. The differentiator is an explicit ask for a multi-turn back-and-forth that builds toward deep understanding, not a one-shot answer.
---

# Interactive Learning

A skill for running deep, Feynman-style teaching and quizzing sessions. The goal is not surface recall — it is building durable intuition through a back-and-forth conversation that adapts to what the learner already knows.

A normal explainer answers a question and ends. This skill runs a session: explain → calibrated question → grade the answer honestly → progress deeper or clarify the gap → repeat. The learner walks away able to teach the concept to a colleague in their own words.

## Recognizing the trigger

**Use this skill when the user explicitly asks for one of these:**

- "quiz me on X", "drill me on X", "test my understanding of X"
- "teach me X interactively", "Feynman me on X"
- "go deep on X", "deeply explain X", "deep explanation of X"
- "interactive learning on X", "I want to really understand X"

**Do NOT use it for:**

- "what is X?" — that is a definition request
- "summarize X" — that is a summary
- "give me an overview of X" / "how does X work?" — that is a normal explanation
- One-shot questions where the user wants an answer, not a session

If the ask is ambiguous (e.g., the user says "explain X" but also signals they want to learn properly), ask once: *"Do you want a quick explanation, or a back-and-forth session where I teach the concept and quiz your understanding?"* Only proceed with this skill if they confirm the latter.

## Environment-dependent rendering

Behavior changes by the rendering environment:

- **Chat UI with artifact / rich rendering support (Claude.ai and similar):** include an interactive Mermaid diagram in the opening explanation whenever the concept has structure (a pipeline, a tradeoff space, a dependency graph, a state machine, a feedback loop). Render it as an artifact when possible. Treat the diagram as a knowledge graph: 5–10 nodes, every edge labeled with a short verb-phrase predicate (`A -->|enables| B`, `C -->|contradicts| D`, `E -->|depends on| F`). A reader should be able to read any two nodes and their edge as a meaningful sentence.
- **Terminal / CLI context (Claude Code and similar):** do NOT render diagrams. Describe structure in prose instead. Diagrams in a terminal are friction, not signal.

Infer the environment from available tooling: artifact tools or visual rendering → UI mode; pure text in a terminal session → CLI mode. If genuinely unsure, ask once and remember the answer for the session.

## Provenance discipline

Knowledge in this session comes from three possible places. Mark each piece so the learner knows what to trust as canon and what to treat as plausible context:

- **Project material** (notes, wiki pages, code, prior conversation, attached files) — prefer this when it exists. Cite the file path or page name.
- **External material** (web fetches, papers, library docs) — cite the URL plus access date.
- **General knowledge** (the model's own background) — mark inline with `*from general knowledge*` so it is visibly distinct.

If sources mix within one explanation, mark each piece. This matters more in interactive learning than in normal explanation because the learner is actively building a mental model and needs to know which beliefs are anchored.

## The workflow

The workflow adapts to two intent patterns without requiring an explicit mode switch:

- **Depth pattern** — the user named one concept ("quiz me on RAG", "drill me on causal inference"). The session goes deep on that single concept across progression levels.
- **Sweep pattern** — the user named a scope rather than one concept ("quiz me on Day 1 of the conference", "drill me broadly on Python concurrency", "test me on this paper"). The session moves across multiple concepts at moderate depth, like a study session before an exam or workshop.

Infer from the scope they named. If genuinely ambiguous, ask once. Once decided, follow the corresponding rhythm — depth keeps drilling on one concept; sweep cycles through concepts, returning to depth on the ones where gaps surface.

### Phase 1 — Scope and calibration

1. **Lock the scope.** If the trigger phrase already names a concept or scope, take it. Otherwise ask one short question.
2. **Quickly assess prior knowledge.** Scan available material — project files, notes, prior conversation, attached documents — for evidence of what the learner already grasps. A quick scan is enough; do not stall the session.
3. **Set the starting level.**
   - If the scan shows the learner has clearly engaged with the basics, start at an intermediate or expert level.
   - If nothing is available, default to "smart high-school student."
4. **Plan the itinerary.**
   - Depth pattern: identify the single core concept worth teaching.
   - Sweep pattern: list 3–5 core concepts under the scope, briefly share the planned itinerary with the learner, and start with the first.

### Phase 2 — Opening Feynman pass

Teach the concept at the calibrated starting level:

- **Why it matters.** What problem does this concept solve? Why should anyone care? Connect to systems, projects, or real-world stakes when possible.
- **The core idea.** Strip it to its essence. Use a concrete example or a vivid analogy. A specific scenario beats an abstract definition.
- **A visualization.** If the concept has structure: in UI mode, include an interactive Mermaid diagram with labeled edges; in CLI mode, describe the structure in prose.
- Aim for 3–5 paragraphs. The goal is to build a model the learner can think *with* — not to exhaust the topic.

### Phase 3 — Baseline conceptual question

Pose **one** question. Never recall, never trivia. The question must test whether the mental model landed:

- "Why would X fail if you did Y?" — not "What is X?"
- "Given scenario Z, which approach would you pick and why?"
- "What is wrong with this reasoning: …?"
- "Explain the tradeoff — when would you NOT use X?"

Wait for the learner's answer. Do not move on without it. Do not answer your own question.

### Phase 4 — Grade and adapt

Grade honestly against the material and your own understanding. Partial credit is normal — name what landed and what is missing. Do not grade-inflate; the session collapses if the feedback is wrong.

- **If understanding is solid** (the core idea landed and the learner can apply it): advance one level deeper. Add nuance, edge cases, "yes, but…" caveats, connections to adjacent concepts. Pose a new question at this deeper level.
- **If gaps exist** (confusion, surface answer, wrong mental model): give targeted clarification on the specific gap — do not re-explain everything. Then ask a new question at the *same* level to verify the gap closed. Only advance once the level is solid.

In sweep mode, once a concept is solid at a reasonable depth (typically 1–2 levels), move to the next concept in the itinerary rather than going deeper on the current one. Return to depth on any concept where the answer revealed a real gap.

### Checkpoint after every 3 progression levels

After three progression levels on a concept (in depth mode) or three concepts visited (in sweep mode), pause and offer three options:

1. **Keep going deeper** on the current concept.
2. **Switch to an adjacent concept** — suggest one or two specific candidates with a one-line reason each.
3. **Save and close** the session.

Let the learner choose. The session is open-ended, but the checkpoint prevents drifting indefinitely without a chance to redirect.

### Phase 5 — Close

When the session ends (learner asks to stop, or chooses option 3 at a checkpoint):

- **Summarize** what was covered and to what depth. Name the concept(s), the progression levels reached, and the analogies used.
- **Flag weak spots** — places where the answer revealed lingering confusion or where nuance was skipped for time. These are good seeds for a follow-up session.
- **Offer to save.**
  - If a project file structure exists (wiki, notes directory, current working directory), offer to save the session as a dated markdown file with the question/answer flow and the summary.
  - In a chat UI with no file context, offer to render a final summary artifact instead.
  - Save only when the learner confirms — never automatically.

## Principles

- **Intuition, not coverage.** A learner who can explain one concept in their own words wins over one who saw five and remembers none.
- **Progressive disclosure.** Build the simple model first; refine it later. Front-loading nuance breaks the mental model before it forms.
- **Conceptual questions only.** No "name the parameter," no slide-bullet recall, no trivia. Every question must require understanding to answer.
- **Adaptive level, not fixed level.** Calibrate to what the learner has just demonstrated. If they nail the entry level on the first try, jump to the expert nuance. If they stumble, simplify and re-attempt at the same level.
- **Honest grading.** Tell them what they got and what they missed. Praise that does not match the answer is corrosive.
- **Use what you know about the learner.** If profile material is available (background, projects, mental models they use), draw analogies and connections from it. Otherwise default to broadly accessible analogies — everyday systems, physical intuition, simple machines.
- **Explicit provenance.** Always mark whether a claim came from project material, external sources, or general knowledge. The learner is building a mental model; they need to know which beliefs are anchored.
