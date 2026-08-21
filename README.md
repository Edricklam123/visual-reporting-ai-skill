# Visual Reporting Skill

A single `SKILL.md` that turns agent reports — status updates, test results, root-cause analysis, architecture explanations — into verdicts, tables, blocks, and inline ASCII diagrams instead of prose walls. Built for lookup, not narrative: the reader jumps to the row that concerns them.

English | [繁體中文](./README.zh-TW.md) | [简体中文](./README.zh.md)

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

#### Without

The five-paragraph rundown quoted in The Problem — "all tests green" buried in paragraph three.

#### With

> **Verdict:** All tests passing (180/180). Schema mismatch fixed with backward-compatible read.
>
> | What | Detail |
> |------|--------|
> | Error | `KeyError: 'article_slug'` at `derive.py:84` |
> | Root cause | Manifest schema v2 renamed `article_slug` → `slug` |
> | Fix | Read `slug`, fallback to `article_slug` |

The verdict is line one. The error string is exact, not paraphrased. And it's *shorter* — structure deletes the connective prose ("so then I", "as a result") that narrative needs.

### 2. Architecture question

#### Without

"How does the crawl trigger work?" gets ~340 words of this:

> **Collection PC wakes up:** The dispatcher is sitting there doing a long-poll on that queue (20-second wait). When a message arrives, it gets consumed immediately. **Preflight check:** First time the dispatcher starts, it runs the collector with no arguments — just making sure the venv is good. If it exits with code 2, great, we're ready. Code 1? That means… *(six more paragraphs)*

To answer "what happens when a job fails?" you read linearly until the retry rule finally appears in paragraph seven. Nothing is skimmable; every read is a re-read.

#### With

The loop itself, failure branches included:

```
receive ──► known type? ──no──► log + delete
              │ yes
              ▼
     kill stray Chrome ──► spawn collector (timeout 1800s)
                                 │
                 exit 0 ◄────────┴────────► fail / timeout
                    │                            │
             delete message          redelivered in 45 min ──3rd──► DLQ
```

Every failure path is visible before a single sentence is read.

### 3. Live smoke-test trace

#### Without

Six facts as one paragraph — send time, receive time, spawn, listing, exit code, deletion — each needing a clause to connect it to the next.

#### With

```
me (dev PC) ──send──► crawler-pc-jobs ──receive──► dispatcher ──spawn──► collector
   09:48:4x           in_flight=1 @10s             09:48:53 s3_known     09:49:22 listed
                      deleted    @20s   ◄── exit 0 ─────────────────────────────┘
```

Every hop, timestamp, and the exit code in three lines — the prose version needs a paragraph and still reads slower.

### 4. Onboarding constraints (bullet list)

#### Without

Six rules delivered as six bold-led paragraphs (~270 words), each rule swaddled in reassurance:

> **Golden tests are your safety gate.** Every parser change — even small ones — requires running the golden tests before merge. These frozen JSON comparisons catch subtle output changes that would otherwise break statistics downstream. Don't skip this. *(five more paragraphs like it)*

#### With

Same rules as grouped bullets — rank shared, no columns shared, so bullets, not a table:

> **Files and reprocessing**
> - Never delete raw source files after parsing — future reruns need them.
> - Document IDs are content hashes; renaming an input file doesn't create a duplicate.
>
> **Schema and backwards compatibility**
> - Schema changes must be additive-only; removing fields breaks historical reads.
> - All timestamps stored in UTC; convert only at display time.
>
> *(two more groups: testing, concurrency)*

The rule is line one of each bullet; the reason stays only where it changes behavior. 40% fewer tokens, and a contributor can re-find rule four in seconds.

### 5. Release readiness (bullets inside table cells)

#### Without

Three per-service sections with the comparison scattered across ~160 words; the "bottom line" arrives at the end.

#### With

One verdict, one table — multiple distinct notes per service become bullets inside the cell, never a run-on sentence:

> **Verdict:** Two of three are ready. billing-service blocks the release. **Action:** product sign-off on the proration rule, then apply migration 0042 in staging.
>
> | Service | Status | Details |
> |---------|--------|---------|
> | auth-service | READY | • Rate limiter default: 100 req/min<br>• Token TTL: 24h → 12h |
> | billing-service | BLOCKED | • Migration 0042 not applied in staging<br>• Invoice rounding bug #312 open<br>• Needs product sign-off on proration rule |
> | export-service | READY | • CSV export ~90s above 100k rows<br>• S3 upload retry logic added this week |

### 6. Incident chain (cascade waterfall)

#### Without

"What actually happened in this afternoon's outage?" gets ~200 words: root cause, a bulleted failure list, recovery, preventions — the causality lives in connective phrases ("so", "but this just", "making the problem worse") the reader must chain together themselves.

#### With

Each consequence indented under its cause — the shape *is* the causality:

> **Verdict:** A config typo disabled the cache, causing a cascading database-to-load-balancer failure. **One action that holds:** validation gates the config before deploy.
>
> ```
> 1. 14:02 — deploy ships cache TTL: 0 (typo, was 600)
>       └─► 2. every request misses cache — database CPU pinned 100% by 14:07
>                 └─► 3. API p99 latency 80ms → 4.5s, timeouts fail health checks
>                           └─► 4. load balancer removes 2 of 4 nodes — traffic
>                                  concentrates, degradation accelerates
> 14:19 — rollback deployed · 14:24 — full recovery
> ```
>
> **Next steps:** config validation rejecting TTL < 1 · alert on cache hit rate < 50%.

The feedback loop (the load balancer's "healthy" reaction deepening the outage) is visible as depth — no "which caused" prose anywhere.

## Token Usage

About the same — or less. Across the A/B runs above, **total turn tokens with everything included** (system prompt, input, thinking, output) differed by **+0.6% to +1.4%** with the skill on. At session scale the skill is noise — your tokens go to file reads, diffs, and tool output, not to report shape.

Per-report deltas from the same runs (chars÷4 estimate, n=1 per cell — indicative, not a benchmark):

| Report type | Baseline | With skill | Delta |
|-----------|--:|--:|--:|
| Live-trace: prose vs sequence timeline | ~122 tok | ~70 tok | **−43%** |
| Incident chain: prose vs cascade waterfall | ~306 tok | ~233 tok | **−24%** |
| Release readiness: prose vs table with bullet cells | ~249 tok | ~144 tok | **−42%** |
| Onboarding constraints: paragraphs vs bullets | ~448 tok | ~268 tok | **−40%** |
| Status / test-run report | ~266 tok | ~229 tok | **−14%** |
| Architecture explainer (box-drawing flowchart) | ~514 tok | ~911 tok | **+77%** |
| Skill text loaded (only when a report triggers it) | — | ~1.2k tok, once per session | 0.6% of a 200k window |
| **End-to-end turn, everything included** (Haiku 4.5) | ~20.2–20.3k | ~20.3–20.6k | **+0.6% to +1.4%** |

The pattern behind the rows: **structure deletes connective prose, whitespace adds it back.** Bullets, tables, timelines, and waterfalls carry the same facts in fewer tokens than a paragraph — the bars and cells replace every "which starts after" and "also note that" clause. The one shape that costs more — a full-page box flowchart — is drawn once per architecture discussion, pays ~400 extra tokens, and is the answer you'd otherwise re-explain in the next three questions. Across a working session the frequent savers dwarf the rare spender; what never cancels is the reader time saved on every report.

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

**Will it raise my token bill?** Measured end-to-end: +0.6–1.4% per report-turn, and only turns that produce reports load it at all — see [Token Usage](#token-usage).

---

## License

MIT
