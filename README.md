# Visual Reporting Skill

A single `SKILL.md` that turns agent reports — status updates, test results, root-cause analysis, architecture explanations — into verdicts, tables, blocks, and inline ASCII diagrams instead of prose walls. Built for lookup, not narrative: the reader jumps to the row that concerns them.

English | [简体中文](./README.zh.md)

## The Problem

Agents answer report-shaped questions with narrative:

> Quick rundown of what I hit and fixed... **The problem:** Ran the full test suite and got 2 failures out of 180 in `parsers/tests/test_feed_derive.py` — both the same root cause. `derive.py` was trying to read `manifest['article_slug']` but the manifest schema got bumped to v2...

Five paragraphs later, "all tests green now" is buried in paragraph three. The reader re-reads the whole thing hunting for the one number that matters — and does it again on every report, every session.

## The Solution

One hard rule — **no wall of text**: a paragraph past ~3 sentences is hiding a shape. Everything else is a menu the agent designs from:

| Content shape | Rendered as |
|-----------|-----------|
| 3+ items sharing attributes | table |
| Homogeneous points, same hierarchy | bullet list |
| Chain that can fail | flowchart |
| Cause + evidence + fix | block |
| Overall state | verdict line |
| Commands, errors, paths | code block, verbatim |
| A single fact | a sentence |

Two guards keep it honest: structure must encode something true (a one-row table is a sentence in disguise), and each finding gets exactly one home — **structure carries facts, prose carries reasoning**.

Full text: [`skills/visual-reporting/SKILL.md`](skills/visual-reporting/SKILL.md). Distilled from real sessions shipping a multi-repo data pipeline, where the same agent reported deploys, IAM audits, parity checks, and live failures every day.

## Before / After

Same model (Haiku 4.5), same input facts — the only difference is the skill.

### 1. Status report

Without the skill: the five-paragraph rundown quoted above ([full sample](benchmarks/samples/status-report.baseline.md)). With it ([full sample](benchmarks/samples/status-report.skilled.md)):

> **Verdict:** All tests passing (180/180). Schema mismatch fixed with backward-compatible read.
>
> | What | Detail |
> |------|--------|
> | Error | `KeyError: 'article_slug'` at `derive.py:84` |
> | Root cause | Manifest schema v2 renamed `article_slug` → `slug` |
> | Fix | Read `slug`, fallback to `article_slug` |

The verdict is line one. The error string is exact, not paraphrased. And it's *shorter* — structure deletes the connective prose ("so then I", "as a result") that narrative needs.

### 2. Architecture question

The baseline answers with nine paragraphs; the skilled answer draws the dispatcher loop as an inline box-drawing flowchart with every failure branch visible, plus a failure-modes table ([baseline](benchmarks/samples/architecture.baseline.md) · [skilled](benchmarks/samples/architecture.skilled.md)).

### 3. Live smoke-test trace

Same six facts as prose ([sample](benchmarks/samples/trigger-trace.prose.md)) versus one sequence timeline ([sample](benchmarks/samples/trigger-trace.timeline.md)):

```
me (dev PC) ──send──► crawler-pc-jobs ──receive──► dispatcher ──spawn──► collector
   09:48:4x           in_flight=1 @10s             09:48:53 s3_known     09:49:22 listed
                      deleted    @20s   ◄── exit 0 ─────────────────────────────┘
```

Every hop, timestamp, and the exit code in three lines — the prose version needs a paragraph and still reads slower.

## Token Usage

About the same. In the A/B runs above, **total turn tokens with everything included** (system prompt, input, thinking, output) differed by **+1.1% and +1.4%** with the skill on. At session scale the skill is noise — your tokens go to file reads, diffs, and tool output, not to report shape.

Per-report deltas, measured on the samples (chars÷4 estimate, n=1 per cell — indicative, not a benchmark):

| Report type | Baseline | With skill | Delta |
|-----------|--:|--:|--:|
| Live-trace: prose vs sequence timeline | ~122 tok | ~70 tok | **−43%** |
| Status / test-run report | ~266 tok | ~229 tok | **−14%** |
| Architecture explainer (box-drawing flowchart) | ~514 tok | ~911 tok | **+77%** |
| Skill text loaded (only when a report triggers it) | — | ~1.2k tok, once per session | 0.6% of a 200k window |
| **End-to-end turn, everything included** (Haiku 4.5) | ~20.3k | ~20.6k | **+1.1% / +1.4%** |

The pattern behind the rows: **structure deletes connective prose, whitespace adds it back.** Tables and dense timelines carry the same facts in fewer tokens than a paragraph. The one shape that costs more — a full-page box flowchart — is drawn once per architecture discussion, pays ~400 extra tokens, and is the answer you'd otherwise re-explain in the next three questions. Across a working session the frequent savers and the rare spender cancel out; what never cancels is the reader time saved on every report.

Want real numbers? The samples and prompts are in [`benchmarks/`](benchmarks/) — PRs with larger-n runs welcome.

## Use Cases

Where it earns its load, all field-tested:

| Use case | What the skill produces |
|-----------|-----------|
| Verification runs | verdict line + PASS/BLOCKED table, not "everything looks good" |
| Root-cause writeups | one Cause / Evidence / Fix block; error strings verbatim |
| Deploy-day status | component × state table the operator walks top to bottom |
| Architecture / flow questions | inline ASCII diagram with failure branches drawn |
| Decision records | Keep/Cut/Change or Assumption/Reality/Consequence tables |
| Audits | "what works / what doesn't" as rows with evidence cells |

Where it stays out of the way: single-fact answers, code deliverables, and conversation stay plain — the skill's own guards say so.

## Install

**Claude Code plugin:**

```
/plugin marketplace add Edricklam123/visual-reporting
/plugin install visual-reporting@visual-reporting
```

Updates flow with the repo: `/plugin marketplace update visual-reporting` re-fetches it.

**Everything else:** ask your AI to copy the skill file to local — e.g. "copy [SKILL.md](skills/visual-reporting/SKILL.md) from Edricklam123/visual-reporting into my skills folder" (`~/.claude/skills/` for Claude Code, `.cursor/rules/` for Cursor — a [ready-made .mdc](.cursor/rules/visual-reporting.mdc) exists — or your agent's rules file). It's one markdown file; the AI knows where it goes.

## How to Know It's Working

- Reports open with a verdict line, not "I ran the tests and…".
- Error strings, SHAs, and counts appear verbatim in cells — "works fine" disappears.
- Teammates start pasting your status tables back at you with their own rows filled in.
- One-line questions still get one-line answers — if everything became a table, the guards aren't being read.

## Customization

The skill is one markdown file — edit it like config:

- **House status chips:** swap `PASS / LIVE / BLOCKED / WAITING` for your team's vocabulary.
- **Shape menu:** add rows your domain repeats (e.g. `metric before/after` for perf work).
- **Diagram budget:** if output tokens matter more than reader clicks, delete the Diagrams section — tables and verdicts carry most of the value.

## FAQ

**Why no lite/full/ultra levels?** The skill already scales itself: its guards collapse structure back to a sentence when the content is one fact. Levels would be a knob nobody turns.

**Does it fight terse-output skills?** No — it governs report *shape*, not verbosity. Pairs well with terseness skills: short words inside tables.

**Will every answer become a table?** The guards exist exactly to prevent that: one-row tables, two-box flowcharts, and structured small talk are named violations.

**Will it raise my token bill?** Measured end-to-end: +1.1–1.4% per report-turn, and only turns that produce reports load it at all — see [Token Usage](#token-usage).

---

## License

MIT
