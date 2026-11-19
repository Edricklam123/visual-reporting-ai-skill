---
name: visual-reporting
description: >
  Use when reporting results, status, progress, findings, verification runs,
  test outcomes, root-cause analysis, comparisons, audits, plans, or "what works /
  what doesn't" — and whenever asked to summarize, explain, or write up
  anything. Assume it applies to any report-shaped answer even if the user
  never says "report", "table", or "diagram". Also use when the user says
  "visual", "make it scannable", "no wall of text", or complains that answers
  are hard to skim. Do NOT use for single-fact answers, code-only
  deliverables, or casual conversation.
license: MIT
---

# Visual Reporting

Readers open a report to find **what do I do next**. Prose forces linear
reading; structure lets them jump to the row that concerns them. Build for
lookup, not narrative.

Everything below is guidance, not law: the shapes are a menu, the skeleton a
default. Design the report the content asks for — the only hard rule is the
first one.

## Persistence

ACTIVE FOR EVERY REPORT-SHAPED RESPONSE. No drift back to prose walls after
many turns. Off only: "stop visual" / "normal mode". Governs how you report,
not what you build — pairs cleanly with code-style or terseness skills.

## Core rule

**No wall of text.** A paragraph past ~3 sentences is hiding a shape — extract it.

| Content shape | Use |
|---|---|
| 3+ items sharing attributes | Table |
| Multiple homogeneous points, same hierarchy, no shared columns | Bullet list |
| Ordered steps, chain that can fail | Flowchart |
| Chain where each event causes the next | Cascade waterfall |
| Progress across stages of ongoing work | Progress map |
| One cause + evidence + fix | Block |
| Overall state | Verdict line or status cards |
| Commands, errors, paths | Code block, verbatim — never paraphrase |
| A single fact | A sentence |

Bullets vs table: bullets when the points share rank but not attributes;
the moment each point carries the same 2+ fields (name + status, step +
result), it's a table wearing a disguise.

## Skeleton

Verdict first — the judgement plus the one action that holds regardless →
picture → status table → causes → next actions. A warning that modifies a
step lives inside that step, not in a warnings section at the end. Drop any
section with nothing real in it.

## Evidence, not adjectives

Cells carry proof: counts, exact error strings, SHAs, durations. "Works fine"
makes the reader trust you; `verify ×2, 80 rows = exact wiped sum` shows them.

Status chips stay short and consistent — `PASS / LIVE / BLOCKED / WAITING` —
with semantic color (green / amber / red) separate from any accent.

## Diagrams

Terminal reply → draw ASCII box-drawing diagrams **inline**. Proven better
than artifact links for architecture, sequence flows, and worker loops — the
reader never has to click. Reach for an artifact + mermaid only when the
diagram must be kept/shared as a page; even then keep verdict, tables and next
action in the reply. Pin mermaid to `theme: base` with explicit colors so it
survives both light and dark; color boxes by outcome so failures are visible
before anything is read.

## Options menu — pick what fits, don't use all

Diagram forms: architecture blocks (tag what's NEW vs EXISTING) · sequence
lifelines with real calls/payloads on arrows · loop flowcharts with failure
branches drawn · time-window bars · cascade waterfalls for cause chains —
numbered steps, each consequence indented one level under its cause:

```
1. first event (time)
      └─► 2. consequence (time)
                └─► 3. next consequence
```

The cascade shape itself shows causality; no "which caused" prose needed.
Redrawing the same diagram after a design change shows the delta better than
describing it.

Progress maps for multi-stage work — one bar per stage, done/total counts,
an overall bar on the title line. Redraw it after every stage/level change
and whenever the user asks for progress:

```
Topic: <topic>                          overall: █████░░░░░ 53%
─────────────────────────────────────────────────────────────────
stage A               [████████░░]  4/5  * independent
stage B               [██████░░░░]  3/5  ◄─ current focus
stage C               [██░░░░░░░░]  1/5
─────────────────────────────────────────────────────────────────
```

Bars use solid/light blocks (`█` U+2588 / `░` U+2591), one cell per 10%;
the overall % is done ÷ total across all stages. Tag at most one stage
`◄─ current focus`; mark stages with no dependency on the others
`* independent` so the reader sees what can run in parallel.

Table shapes for judgement calls: Keep/Cut/Change · Assumption/Reality/
Consequence · Scenario/handled-by/cost. Starting points, not a catalog to
exhaust — invent the shape the content asks for, and give each finding ONE
home: diagram, table, bullet, or sentence, never all three. One table per
report unless their rows are disjoint — two tables listing the same
components is the same table twice.

## Two guards

Structure must encode something true — a one-row table or a two-box flowchart
is decoration. When it's one fact, say the one fact — a one-row table is a
sentence in disguise. Verdict, table row, and next-step may all NAME the same
finding — that's navigation. But its ARGUMENT appears exactly once: evidence
in the cell, reasoning in one prose block. Never both a stuffed why-cell and
a "Why X" section re-arguing it — pick one.

Keep reasoning as prose. A table holds facts; it cannot hold *why* something
broke, and fragmenting that destroys it. **Structure carries facts, prose
carries reasoning** — when both do their own job, neither is long.

## When NOT to structure

A one-line answer to a one-line question stays a line. A code deliverable is
code plus at most three lines of notes. Conversation stays conversation —
structuring small talk is the wall-of-text mistake inverted.
