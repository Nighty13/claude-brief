# One-paste install

Paste the block below to your Claude Code agent. It installs both skills and
nothing else — two markdown files, no scripts, no packages, no network calls
beyond fetching this repo.

---

```
Install the claude-brief skills into this project.

1. Fetch these two files:
   https://raw.githubusercontent.com/Nighty13/claude-brief/main/skills/brief/SKILL.md
   https://raw.githubusercontent.com/Nighty13/claude-brief/main/skills/brief-calibrate/SKILL.md

2. Write them to ~/.claude/skills/brief/SKILL.md and
   ~/.claude/skills/brief-calibrate/SKILL.md, creating the directories if
   needed, so they are available in all my projects. Use this project's
   .claude/skills/ instead if I say I only want them here.

3. Do not modify the contents. Do not run anything else. Show me the two
   descriptions when you are done, and tell me that /brief turns the style on
   for a session, and that /brief-calibrate writes this project's own rules to
   .claude/brief-project.md once I have a few real sessions of history here.
```

---

## If you would rather not fetch anything

Open `skills/brief/SKILL.md` and `skills/brief-calibrate/SKILL.md` in this
repo, copy each one, and paste it to your agent with:

```
Write this verbatim to .claude/skills/<name>/SKILL.md, creating the
directories if needed. Do not change the contents.
```

Both files are plain markdown instructions. Read them before installing — they
are short, and you should know what you are putting in your agent's context.

## What to do next

`/brief` works immediately.

`/brief-calibrate` needs history to read. Give it a handful of real sessions in
the project first, then run it. It will show you proposed rules with quoted
evidence from your own transcripts and wait for approval before writing
anything. If it finds fewer than three rules that clear its evidence bar, it
changes nothing and tells you the project's sessions are already tight.
