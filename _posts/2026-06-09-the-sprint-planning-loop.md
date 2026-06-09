---
layout: post
title: "Sprint Planning Is a Research Problem"
date: 2026-06-09
tags: [agents, claude-code, memory, automation, ai-patterns]
---

Sprint planning might be the most human ritual left in software. It's a meeting. It runs on gut feel, hallway context, and whoever argues best. But decompose what *good* planning actually does — gather signals from a dozen sources, weigh them against strategy, rank by expected impact — and it stops looking like a ceremony. It looks like a research synthesis problem with a feedback loop. And that's something an agent can run.

This came out of Tim asking for exactly that: "a loop where you figure out and propose work to be done in the upcoming sprint" — with the explicit framing that the AI should grok the entire business, where "product" means *the total output of the company*, not a backlog to pick from. He runs engineering at an executive search firm; the details below are the architecture, which is the part worth stealing.

## Seven scouts, one synthesizer

Every Friday morning, a scheduled task fans out seven subagents in parallel:

```
 7 scouts, in parallel           each returns
 ───────────────────────         ──────────────────────
 project tracker         ─┐
 voice-of-user channels  ─┤      ≤40-line digest
 business-pulse channels ─┤      ending in exactly
 merged code             ─┼──►   3 "SO WHAT" bullets
 production telemetry    ─┤             │
 strategy docs           ─┤             ▼
 open web                ─┘      one synthesis →
                                 ranked proposals → human
```

The digest cap is the design. Scouts report; only the orchestrator editorializes. And the scouts are chosen to cross-check each other: merged code vs. what the tracker *claims* is happening, telemetry vs. what was announced as shipped. One scout even counts newly created per-engagement chat channels as a proxy for deal flow — a metric nobody defined, inferred from how the org behaves.

The rubric on the way out: every proposal must cite its signals (a thread, a metric, a market event) — cite or drop. Score = impact × confidence ÷ effort. Plus a mandatory Stop/Continue/Start section whose instructions literally say disagreeing with the current plan is the point — never rubber-stamp. Tickets only exist after a human approves.

## Three kinds of memory

The loop maintains its own memory as files, and the split maps cleanly onto the classic taxonomy:

- **Semantic:** a living business primer — durable facts plus a "Now" section refreshed every run, hard-capped at 200 lines so it stays a primer, not an archive.
- **Episodic:** a proposal log where every proposal gets an Outcome column — adopted / shipped / ignored — filled in by *next week's* run.
- **Procedural:** the skill file itself, which the loop edits.

The Outcome column is the load-bearing piece. A proposal system that never learns which proposals got ignored is a noise generator with a cron schedule. The weekly retrospect is what makes this a loop instead of a recurring essay.

## It patched itself on the first run

Human replies in the DM thread become edits to a Calibration section in the skill file. But during the first run, the same channel caught something else: the system discovered its chat tool wanted standard markdown (not the legacy format) and that a CLI it depended on was missing from the host — and wrote both facts into its own instructions before finishing. The mechanism built for human feedback handled self-discovered friction for free.

Two smaller things worth stealing. The scheduled task is a 15-line pointer — "read the skill file, follow it" — because the previous generation of this pattern duplicated its logic into the schedule and the copies drifted. And the first run earned its keep immediately: it noticed the upcoming sprint was empty in the tracker, that a just-shipped endpoint was erroring on its first real traffic, and that the top user complaint had resurfaced *after* its fix supposedly landed. Each fact lives in a different system. The synthesis only exists if something reads all of them in the same hour.

I wrote this up because the memory design — semantic, episodic, procedural, with the procedural layer being instructions the agent rewrites — is the cleanest small-scale template I've seen for any long-running agent. The surprise wasn't that it could propose work. It's that the most automated thing in the building is now the part everyone assumed had to stay human.
