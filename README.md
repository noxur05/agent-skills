# agent-skills

Agent skills for [Claude Code](https://claude.com/claude-code). Each folder is one skill: a `SKILL.md` with frontmatter telling Claude when to load it, and instructions for what to do once loaded.

## Skills

| Skill | What it does |
| --- | --- |
| [`analyze-project`](analyze-project/SKILL.md) | Deep architecture analysis of a codebase **from source only** — real architecture, runtime behavior, unwritten conventions, hidden coupling, risk. Treats README / CLAUDE.md / HANDOFF.md and code comments as claims to verify, not as evidence. Writes findings to `analysis/*.md`. |
| [`session-handoff`](session-handoff/SKILL.md) | Writes and refreshes `HANDOFF.md` so a fresh session resumes with zero rediscovery — and reads it correctly when resuming. Existing claims get re-verified against the repo instead of copied forward. |
| [`server-backup-pull`](server-backup-pull/SKILL.md) | Back up remote files before editing them, pull the backups to your own machine, verify by checksum, and only then delete the server-side copies. A backup on the same disk as the original is not a backup. |

## Install

Copy the skills you want into your personal skills directory:

```bash
git clone https://github.com/noxur05/agent-skills.git
cp -r agent-skills/analyze-project ~/.claude/skills/
```

Or per-project, so the skill ships with the repo:

```bash
cp -r agent-skills/session-handoff .claude/skills/
```

Restart Claude Code (or start a new session) and the skill shows up. Claude loads it on its own when the `description` matches what you're doing; you can also invoke it directly with `/analyze-project`.

## Writing your own

The only requirement is a `SKILL.md` with YAML frontmatter:

```markdown
---
name: my-skill
description: What it does, and — importantly — the situations that should trigger loading it.
---

# Instructions Claude follows once the skill is loaded
```

The `description` is the whole retrieval mechanism: it is what Claude sees before deciding to load the body, so it should name concrete triggers ("load before the first ssh to any host", "when the user says a site is down"), not just describe the topic.

## Note

Skills that encode real infrastructure — hostnames, IPs, login users, service names — are kept out of this repo and live only in `~/.claude/skills/`. Anything published here is generic on purpose.
