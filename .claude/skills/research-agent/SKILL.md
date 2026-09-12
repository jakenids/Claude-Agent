---
name: research-agent
description: Run a gated research workflow on a topic — clarify scope/audience/depth, present a plan inline for approval, research once approved, then save a structured report to output/ as both Markdown and .docx. Use when the user invokes /research-agent, or asks for a researched report, briefing, literature scan, or landscape review on a topic.
---

# Research Agent

A four-phase workflow: **Clarify → Plan → Research → Report.** Each phase gates
the next. Do not skip ahead, and do not collapse phases to save time.

The topic arrives as the argument (`/research-agent <topic>`). If no topic was
given, ask for one and stop until you have it.

---

## Phase 1 — Clarify

Ask **one** `AskUserQuestion` call containing all four questions below. Do not
drip-feed them across turns.

Write the options fresh for *this* topic — never reuse the generic examples
here verbatim. Good options name real boundaries the user would recognize
("post-2022 transformer work only", "including the pre-LLM literature").

1. **Scope** — the boundary. What's in, what's out, what time window.
2. **Audience** — who reads this. Governs vocabulary, what gets assumed,
   whether jargon is defined on first use.
3. **Depth** — one of:
   - `scan` — ~5 sources, skim-level, headline findings only
   - `brief` — ~15 sources, every claim traced (**default** if unanswered)
   - `deep` — ~30+ sources, competing positions weighed against each other
4. **Angle** — neutral survey / decision input / comparison. This reshapes the
   outline more than the other three and is the easiest to get wrong silently,
   so always ask it even when the topic seems obvious.

If the user skips a question, pick the default, and **say in one line which
default you picked** before moving on.

---

## Phase 2 — Plan

Present the plan **inline in chat**. Do not write it to a file and do not make
the user open anything to review it. It contains:

- **Framing question** — the single question the report answers. One sentence.
- **Section outline** — 3–7 sections, each with a one-line statement of what it
  establishes. This outline becomes the research fan-out and the report's
  headings, so get it right here.
- **Source strategy** — the kinds of sources you'll pursue (primary literature,
  vendor docs, benchmarks, practitioner writing) and any you'll distrust.
- **Explicitly out of scope** — what you will not cover. Prevents silent drift.
- **Estimated effort** — rough source count and wall-clock.

Keep it scannable — headings and bullets, not paragraphs. It has to be
reviewable in the terminal without scrolling back and forth.

Then **stop and wait.** Do not research before the user approves. If they ask
for changes, re-present the revised plan inline; approval is on the current
version, never on an earlier one.

---

## Phase 3 — Research

Scale to the chosen depth:

- **scan** — do it yourself, serially. No subagents; the overhead exceeds the
  benefit at this size.
- **brief / deep** — fan out parallel `Agent` subagents, **one per outline
  section**, all launched in a single message. This skill authorizes that use
  of subagents.

Give every subagent: the framing question, its section's remit, the audience
and depth, and an instruction to return findings as claim + source URL pairs
with no prose padding. Tell each one what the *neighbouring* sections cover so
they don't duplicate each other's ground.

Reconcile the returns yourself — that step is where contradictions between
sources actually surface, and it does not delegate.

Non-negotiable rules:

- Every non-obvious claim carries a source URL. No source, no claim.
- Where credible sources disagree, **the report says they disagree.** Never
  silently pick a winner.
- Anything you could not verify gets labelled as unverified, not dropped and
  not quietly softened.
- Distinguish what the sources say from what you infer. Mark inference.
- Note the recency of key sources. A 2019 benchmark is not current evidence.

---

## Phase 4 — Report

Produce **two files**, same basename, both in `output/`:

1. `output/YYYY-MM-DD-<topic-slug>.md` — the source of truth. Use
   `references/report-template.md` as the skeleton.
2. `output/YYYY-MM-DD-<topic-slug>.docx` — generated from the finished
   Markdown by invoking the **`docx` skill**. Write the Markdown first and get
   it right; the .docx is a rendering of it, never a separate draft that can
   drift.

### Formatting the .docx for reading

The .docx is the version that gets forwarded and read on a screen or printed,
so it is typeset, not dumped:

- **Real Word heading styles** (Heading 1/2/3), not manually bolded text —
  they drive the navigation pane and the table of contents.
- **Title block** at the top: topic as the title, then the metadata table
  (date, framing question, scope, audience, depth, source count).
- **Table of contents** after the title block for `brief` and `deep` reports.
  Skip it for `scan` — it's longer than the report.
- **Body text** at 11pt in a serif face with ~1.15 line spacing and space
  between paragraphs. Long lines of dense 10pt sans-serif are the single
  biggest readability failure here.
- **Page numbers** in the footer, and the topic in the header.
- **Executive summary bullets** kept as a bulleted list, not run into prose.
- **Sources** as a numbered list, URLs visible as text — a printed page with
  invisible hyperlinks loses the citations entirely.
- Tables get header-row shading and real borders. Comparisons belong in tables
  rather than in parallel bullet lists.
- Page breaks before major sections in `deep` reports only; in shorter ones
  they waste paper.

### Then

In the terminal, give a **short** summary: the three or four findings that
would change the reader's mind, and the paths to both files. Do not paste the
report into chat — it's in the files.

Commit both files to the current branch. This environment is ephemeral and
uncommitted output is lost when the container is reclaimed. Push only if the
user asks.

---

## Notes

- Topic slug: lowercase, hyphenated, ~5 words max (`quantum-error-correction`).
- If mid-research the topic turns out to be materially different from what the
  plan assumed, stop and say so rather than quietly researching something else.
- If the topic is one where sources are mostly marketing (vendor comparisons,
  emerging products), say so in the report's limitations rather than
  presenting vendor claims as findings.
