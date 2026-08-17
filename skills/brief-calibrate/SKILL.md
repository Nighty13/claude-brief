---
name: brief-calibrate
description: >
  Derive project-specific compression rules from this project's real session
  transcripts and rewrite the generated block of the `brief` skill. Use when
  the user runs /brief-calibrate, asks to tune or re-tune brief mode, or asks
  what padding this project's sessions actually contain.
---

Find the padding *this* project's sessions actually contain, prove it with
quotes, and rewrite the generated block of `.claude/skills/brief/SKILL.md`.

Never touch anything outside the BEGIN/END GENERATED markers.

## 1. Protected first

Before reading any transcript, read the project's `CLAUDE.md` and the user's
memory index. List everything the project has explicitly asked for — proactive
flagging, terminology definitions, evidence requirements, judgement handoffs.

That list is protected. No generated rule may suppress, shorten, or make
optional anything on it. If the evidence points at a protected behavior,
discard the finding and say so in the report. Volume is not padding when the
user asked for the volume.

## 2. Sample the evidence

Transcripts live in `~/.claude/projects/<cwd-slug>/*.jsonl`, one line per
event. The directory is large — hundreds of MB is normal. Do not read whole
files into context.

Take the 10 most recent by mtime, excluding the current session — a session
about the skill will poison the sample. Extract assistant text blocks only,
never tool results, never user messages:

    ev["type"] == "assistant" → ev["message"]["content"][] where type == "text"

Write them to a scratch file, then analyze with counting scripts. `jq` may not
be installed; Python is. On Windows set `PYTHONIOENCODING=utf-8` or emoji in
the transcripts will crash the console writer.

Verify the shape on one file before trusting the filter — the schema drifts
between versions. If it returns nothing, inspect a line and adjust rather than
proceeding on empty input.

## 3. Measure, do not eyeball

Report the baseline first: turn count, session count, median and p90 turn
length. A project whose median turn is already short does not have a verbosity
problem, and the rules should target habits instead.

Then count candidate patterns per session:

- Turn-initial bridge words and openers.
- Narration of this project's specific tooling.
- Agreement and self-correction openers.
- Markdown decoration density — bold spans per turn, emoji, headers on short
  answers.
- Fenced blocks over ~25 lines.
- Re-explaining project vocabulary the user demonstrably knows.

## 4. Promotion bar

A finding becomes a rule only if all four hold:

1. It appears in **at least 2 different sessions**.
2. You can quote a **verbatim example** pulled from a transcript.
3. Cutting it would not have cost the user information.
4. It is not on the protected list.

Point 3 is where this fails if you rush it. Pull real quotes and read them
before promoting a word. A word that looks like filler in the abstract is
often load-bearing in context — check what it is doing in the sentence, not
how often it appears.

Anything that fails the bar is an observation. Report it, do not encode it.

## 5. Rewrite, do not append

Re-derive the entire generated block from scratch each run. Do not read the old
block before deriving — derive, then diff.

Ceiling: **12 rules, 40 lines.** This text is injected every turn; past that it
costs more input than it saves in output. At the cap, keep the rules with the
most occurrences and drop the rest.

Each rule is one line: the rule, then the count, then a short real quote.
Counts age — they tell the next run whether the habit survived.

## 6. Confirm before writing

Show the user:

- The proposed block.
- A diff against the current one.
- Every rule dropped since last run, with the reason: no longer observed,
  displaced at the cap, or now protected.
- Every finding discarded for touching a protected behavior, with its count —
  these are the near-misses, and they are the most useful part of the report.

Wait for approval. Then replace only the text between the markers.

## Honest scope

This tunes assistant prose, which on most engineering sessions is a minority of
the tokens — file reads, tool results, and diffs dominate. Expect a modest
saving and a real gain in signal density.

If a run produces fewer than three rules, say so and change nothing. A project
whose sessions are already tight does not need a generated block, and
manufacturing rules to justify the run is the failure mode of this skill.
