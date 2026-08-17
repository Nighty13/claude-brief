# brief

Two skills for Claude Code. One makes responses shorter without making them
worse. The other reads your project's own session transcripts and works out
what *your* sessions specifically waste words on.

Generic "be concise" prompts cut the same twelve filler words from every
project. The padding in a Rust engine repo is not the padding in a Django app,
and a rule written from guesswork deletes words that were carrying meaning.
This measures instead.

## Install

Easiest: open [PROMPT.md](PROMPT.md) and paste the block to your Claude Code
agent. It writes the two files for you.

By hand, per project:

```bash
git clone https://github.com/Nighty13/claude-brief .tmp-brief
cp -r .tmp-brief/skills/* .claude/skills/
rm -rf .tmp-brief
```

For every project, copy into `~/.claude/skills/` instead.

No npm package, no CLI, no install script, no hooks. Two markdown files, and
you should read them before installing.

## Use

Type `/brief` once at the start of a session. It stays on until `/brief off`.

Then, after you have a few real sessions in the project, run
`/brief-calibrate`. It samples your transcripts, proposes project-specific
rules with quoted evidence, shows you a diff, and waits for approval before
writing anything.

Do not put these rules in `CLAUDE.md` to avoid typing `/brief`. `CLAUDE.md`
loads every turn of every session, so you would pay the rule's input cost
permanently to save output tokens occasionally. On-demand is the point.

## How calibration avoids the obvious traps

A model auditing its own transcripts is a biased instrument. Asked to find
padding, it will find padding — including in sentences that were doing work.
Three defenses are built in:

**Evidence bar.** A finding becomes a rule only if it appears in at least two
different sessions, can be quoted verbatim, and cutting it would not have cost
the reader information. Everything else is reported as an observation and not
encoded.

**The file is split.** A fixed core that calibration never touches, and a
generated block between markers that it rewrites wholesale. Each run
re-derives from scratch, so rules that no longer earn a citation die instead
of accumulating. Cap of 12 rules — past that the skill costs more input per
turn than it saves in output.

**A protected list.** Before reading a single transcript, calibration reads
your `CLAUDE.md` and memory files and lists everything the project explicitly
asked for. No generated rule may suppress anything on it. Volume is not padding
when you asked for the volume.

There is a fourth defense in the core skill, and it is the one that matters
most in practice:

> Do not do less work. This shapes prose, not effort: same files read, same
> checks run, same edge cases handled. A short answer that skipped a step is
> not brief, it is wrong.

Compression prompts fail by quietly turning "don't say X" into "don't do X".
The output looks fine. That is what makes it dangerous.

## What a real run found

On a ~100k-line Rust/WGSL engine codebase, across 2,778 assistant turns from
10 sessions:

Median turn was already 114 characters. The sessions were not verbose, so the
promoted rules targeted habits instead — a turn-initial bridge word appearing
in 423 of 2,778 turns and carrying no information, narration lines that only
announced the next tool call, and 3,524 bold spans across 2,778 turns diluting
emphasis to nothing.

The interesting result was what it **refused** to promote. The single most
frequent candidate in the entire corpus was the word "actually" — 283 hits,
present in all 10 sessions. Every generic concision prompt deletes it. The
quotes said don't:

- "verify the stub *actually* runs"
- "which gate *actually* catches the bug"
- "*actually* makes every arrow point downward, rather than assuming it does"

In that project "actually" was not filler. It was the word separating
*verified* from *assumed* — load-bearing in a codebase whose house rule is to
measure rather than infer. Same for "just" (166 hits), which almost always
meant "only", where dropping it inverts the claim.

A generic filler rule would have stripped 449 meaning-carrying words from that
project. That result is the entire argument for measuring before cutting.

## Honest scope

This tunes assistant prose, which on most engineering sessions is a minority of
the tokens — file reads, tool results, and diffs dominate. Expect a modest
token saving and a real gain in signal density. If you want a large saving, the
lever is what gets read into context, not how the answer is phrased.

No percentage is claimed here, because the honest number depends on your
workload and the rules themselves cost input tokens on every turn they are
loaded. Calibration reports fewer than three rules? It changes nothing and says
so. A project whose sessions are already tight does not need this.

## Credit

The idea of a persistent compression skill, and several of the sharper rules —
notably that invented abbreviations (`cfg`, `impl`, `fn`) save nothing because
the tokenizer splits them the same as the full word — come from
[JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) (MIT).

This is an independent rewrite with different goals: no persona, no intensity
dial, no hosted gateway, no hooks, and per-project calibration from real
evidence rather than one fixed ruleset. If you want the caveman voice, use
caveman — it is a good skill and it is funnier than this one.

## License

MIT.
