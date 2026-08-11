---
name: session-handoff
description: Write or refresh HANDOFF.md so a fresh Claude session can resume work with zero rediscovery — and read it correctly when resuming. Load when the user says they are wrapping up, asks for a handoff, says "where were we", "what has been done so far", "continue where we left off", "are we done", before a /clear or /compact, or when a session has run long enough that context is about to be lost.
---

# Session handoff

Two directions. Pick the one that matches.

---

## Direction A — Writing the handoff

The next session has **no memory** except `HANDOFF.md` and the repo contents. Optimise for operational continuity, not readability.

### Gather evidence first — do not write from conversation memory alone

```bash
git status && git log --oneline -20 && git diff --stat
git stash list
ls -la && cat package.json 2>/dev/null | head -40   # or Cargo.toml / pyproject.toml
grep -rn "TODO\|FIXME\|XXX\|HACK" --include='*.rs' --include='*.ts' --include='*.tsx' --include='*.py' . | head -40
ls .github/workflows/ deploy/ migrations/ 2>/dev/null
cat .env.example 2>/dev/null
```

Never read `.env`, `*.pem`, `*.key`, or service-account JSON. Use `.env.example` for the variable list.

Also inspect: recently modified files (`git log --name-only -10`), startup scripts, systemd/nginx/docker configs, and CI definitions.

### Rules

- No vague summaries. "Backend mostly complete" is worthless — say which endpoints exist, which are stubbed, which are untested.
- Exact filenames, exact commands, exact paths, exact env var names.
- Include **failed attempts and abandoned approaches**, with why they failed and whether partial code survives. This is the single highest-value section — it is what stops the next session re-walking a dead end.
- Include bugs introduced this session, resolved or not.
- Include every correction the user made, and the approach finally accepted.
- Include things the user explicitly said **not** to do.
- Mark speculation as speculation. Prefer "unverified — assumed from naming" over a confident wrong claim.
- Bullets over paragraphs. Technical precision over fluff.
- Secrets are `<REDACTED>`, never real values.
- Date the document and date individual claims that could go stale.

### Structure

```
# HANDOFF.md
_Last updated: YYYY-MM-DD — branch <name> @ <short-sha>_

## 1. Project Overview
purpose, current stage, high-level architecture, stack, deploy targets, external services

## 2. Current Objective
what was in flight, what was supposed to happen next, next tasks in priority order, blockers

## 3. Repository Structure
important dirs/files with why they matter; entry points; infra and scripts

## 4. Environment Setup
required runtimes and versions; install / dev / build / test / lint / deploy commands, verbatim

## 5. Environment Variables
name, purpose, required or optional, example format. Secrets as <REDACTED>.

## 6. Backend Architecture
request flow, auth, authorization, DB layer, background jobs, caching, events, middleware,
external integrations — with the exact files where the logic lives

## 7. Frontend Architecture
routing, state management, API layer, auth flow, UI libs, component organisation, build system

## 8. Database State
engine, migration mechanism and current state, important tables and relationships,
known schema issues, pending migrations

## 9. Infrastructure & Deployment
hosts, domains, reverse proxy, TLS, systemd/docker/pm2, CI/CD, cron, logging, backups, firewall

## 10. Current Bugs / Known Problems
per issue: description, probable cause, attempted fixes, status, severity, relevant files

## 11. Failed Attempts & Rejected Approaches
per attempt: what, why it seemed reasonable, why it failed, error symptoms,
whether partial code remains, retry or avoid

## 12. User Constraints / Warnings / Preferences
prohibited actions, infra constraints, deploy preferences, resource and budget limits,
style and tooling preferences, security concerns, UX expectations, repeated corrections
— THIS SECTION IS CRITICAL

## 13. Pending TODOs
- [ ] ordered by priority

## 14. Recent Important Decisions
architectural decisions and WHY, tradeoffs, temporary hacks, rejected alternatives

## 15. Git Status
branch, modified files, uncommitted work, risky changes, stashes, notable recent commits

## 16. Critical Files To Read First
exact path + why it matters, ordered by importance

## 17. Suggested Next Prompt
the exact prompt to paste into the next session

## 18. Session Notes
debugging commands that worked, important log locations, environment quirks,
partial implementations, workarounds, easy-to-forget gotchas

## 19. Inferred / Uncertain Information
assumptions, unverified architecture details, areas needing validation next session
— clearly separated from confirmed fact
```

### Before finishing

Ask yourself: what operational context is missing, what assumptions are hidden, what unresolved uncertainty is unrecorded, and what would waste the most time if forgotten. Add those.

### Refreshing an existing HANDOFF.md

Do not blindly overwrite. Re-verify each existing claim against the current repo, correct what drifted, and mark corrected entries with the new date. A HANDOFF.md that quietly keeps a stale claim is worse than none.

---

## Direction B — Resuming from a handoff

Read `HANDOFF.md` **and** `CLAUDE.md` before touching code. Then:

1. Check §16 (critical files) and read them.
2. Check §11 (failed attempts) so you do not repeat abandoned work.
3. Check §12 (user constraints) and treat it as binding.
4. **Verify before acting.** Treat the handoff as a lead, not as truth — it describes the repo as of its date. Confirm any claim you are about to build on: `git log` since that date, does the named file still exist, does the command still run. Sections marked in §19 are explicitly unverified.
5. State what you verified and what you are taking on trust before you start editing.
