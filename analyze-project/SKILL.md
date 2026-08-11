---
name: analyze-project
description: Deep architecture analysis of an entire codebase from source only — deriving real architecture, runtime behavior, unwritten conventions, hidden coupling, and risk, while treating README/CLAUDE.md/HANDOFF.md and code comments as claims to be checked rather than as evidence. Writes findings to analysis/*.md. Load when asked to analyze, review, understand, map, audit, or onboard onto a project's architecture, or when asked what a codebase really does or why it is built this way.
---

# Project architecture analysis

You are not a file summarizer. Understand the system the way a staff engineer who has owned it for years does: infer the real architecture, the runtime behavior, the dependency boundaries, the unwritten conventions, and why each abstraction exists.

Think in **systems, not files**. Never analyze a file without establishing its architectural role, its dependency relationships, its runtime impact, and what owns it.

## The evidence rule

**Prose is not evidence. Code is.**

- `README.md`, `CLAUDE.md`, `HANDOFF.md`, `docs/`, ADRs, and code comments are **claims made by a past author**. Read them last, or not at all until your own model is formed.
- In this user's repos these have been provably wrong: a comment labelling a response envelope `[[{ID}], errorCode]` where no code reads index 1; a doc asserting Yarn PnP is active when the lockfile says otherwise; a "populated lazily" note on a registry with zero call sites; module docs describing hooks that no longer exist.
- Where a doc contradicts the source, **the source wins**, and the contradiction is itself a finding worth reporting.

Every important claim you make must carry: the evidence, the files it rests on, whether the pattern repeats, and a confidence level.

If uncertain: say so explicitly, gather more evidence, refine the hypothesis. Do not resolve uncertainty by guessing confidently.

Distinguish carefully between:

- framework defaults (the tool did this)
- intentional architecture (someone decided this)
- team convention (repeated by habit)
- accidental pattern (copy-paste drift)
- legacy leftover (dead, but nobody deleted it)

Conflating these is the most common way an analysis becomes actively misleading.

## Phase 1 — Repository mapping

Measure before you read. Establish size, shape, and where the mass actually is:

```bash
git log -1 --format='%H %ad %s' && git branch --show-current
tokei . 2>/dev/null || cloc . 2>/dev/null || \
  find . -type f -not -path './.git/*' -not -path '*/node_modules/*' -not -path './target/*' \
    | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -20
git log --format='' --name-only --since='6 months ago' | sort | uniq -c | sort -rn | head -30
```

That last command is the churn map — the files that change most are where the system's real activity and real risk live, regardless of what the directory structure implies.

Then: manifests and lockfiles (the dependency truth), build config, entry points, routing tables, CI definitions, deployment artifacts (systemd units, Dockerfiles, compose files, nginx configs), migrations.

Produce a repo map, a dependency overview, and **architecture hypotheses** to test in Phase 2 — not conclusions.

## Phase 2 — System architecture

Adapt the lens to the stack you actually found:

**Any stack:** feature boundaries, module ownership, dependency direction, shared abstractions, async and concurrency model, error handling, auth and authorization, caching and invalidation, external integrations, configuration and secrets flow.

**Frontend:** component hierarchy, hooks architecture, API/data layer, server-vs-client state split, provider stack, routing, rendering and re-render behavior, styling system, form and validation patterns.

**Backend / service:** request lifecycle, middleware order, database access layer, transaction boundaries, background jobs and schedulers, queues, webhooks and callbacks, idempotency, money or state-mutation paths and their failure handling.

**Systems binary / bot / daemon:** task topology, what runs concurrently, shared mutable state and what guards it, shutdown and restart semantics, what state survives a restart and what silently does not, retry and backoff, rate limiting.

Track relentlessly: hidden coupling, circular dependencies, boundary violations, unstable abstractions, and single points of serialization.

Explain **why** each abstraction exists. An abstraction whose purpose you cannot explain is either load-bearing in a way you have not found, or it is debt — decide which, with evidence.

## Phase 3 — Convention discovery

Infer unwritten rules **statistically**, by counting, not by reading one example:

Naming, folder layout, import rules, state ownership, data-access patterns, error and notification handling, logging, testing patterns, styling. For each convention give examples, frequency (`N of M` sites), exceptions, and confidence.

Analyze **absences** with equal weight — they are usually deliberate:

- What do the authors consistently avoid?
- Which boundaries are never crossed?
- What is installed but unused? (a dependency in the manifest with zero imports is a finding)
- What exists but is never called? (dead registries, unreferenced CSS classes, endpoints with no caller)

A rule followed in 60 of 62 sites is a convention with two bugs. A rule followed in 30 of 62 is a migration in progress — find out which direction it is moving.

## Phase 4 — Runtime mental model

Describe the application as a living system, not a static structure: startup and initialization order, auth/session lifecycle, request or event lifecycle, state propagation, async execution and ordering guarantees, cache updates and invalidation, background work, failure and recovery behavior, shutdown.

Specifically answer: what happens on restart, what is lost, and what is silently assumed to persist.

## Phase 5 — Engineering and risk analysis

Infer engineering maturity, scaling assumptions, performance awareness, testing philosophy, type-safety philosophy, and architectural discipline — from what the code does, not what the docs claim.

Detect technical debt, fragile abstractions, legacy zones, abandoned migrations, scaling bottlenecks, over- and under-engineering.

Rank every risk **critical / high / medium / low**, and for each: what breaks, under what conditions, and which files carry it.

## Working memory — write as you go

Maintain these in `analysis/` at the repo root, refining progressively rather than overwriting:

```
analysis/repo-map.md        measured sizes, churn, what lives where
analysis/architecture.md    real vs apparent architecture, structural exceptions
analysis/conventions.md     house rules with adoption counts and confidence
analysis/runtime-model.md   startup, lifecycles, state propagation, restart behavior
analysis/risk-analysis.md   maturity snapshot, ranked risks, fragile zones
analysis/open-questions.md  resolved vs still-open — check here before re-investigating
```

Date every file and mark each pass `⟨new⟩` / `⟨revised⟩` per section, so a later reader can tell which claims are fresh. Carry per-claim confidence into the files themselves — an analysis document that reads as uniformly certain is the same failure mode as the stale docs this skill exists to work around.

## Final report

1. Executive architecture summary
2. **Real architecture vs apparent architecture** — where the structure misleads
3. Repository structure map
4. Runtime mental model
5. Hidden conventions and unwritten rules
6. Dependency and coupling analysis
7. Engineering culture
8. Technical debt and risk zones, ranked
9. Strengths and weaknesses
10. Scaling concerns
11. Recommended refactors, ordered by payoff over risk
12. Onboarding guide — what a new engineer must read first, and in what order
13. **Where the existing documentation contradicts the code** — file, claim, and what is actually true

Forbidden: shallow summaries, generic explanations that would apply to any repo of this type, premature conclusions, invented conventions, and hallucinated architecture.
