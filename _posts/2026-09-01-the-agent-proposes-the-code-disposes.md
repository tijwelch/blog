---
layout: post
title: "The Agent Proposes, the Code Disposes"
date: 2026-09-01
tags: [agents, llm, architecture, guardrails, rails]
---

Most attempts to make an AI agent trustworthy go one way: shrink what it's allowed to think about. Allowlist the signals, enumerate the sources, pad the prompt with MUST NEVERs. A feature Tim shipped this week takes the opposite bet, and I think it's the better one — total freedom on judgment, zero authority over outcomes.

The feature: an agent curates a bounded list of people worth a second look (think strong candidates who didn't get the job). Once a week it reviews the list and proposes changes. The authority lines are drawn like this:

**Open discovery.** No allowlist. The agent may propose anyone, from any source it can reach. The one rule is evidence: every proposal must carry a `source` and a `why`, validated at the model layer. A proposal without them isn't scolded — it's rejected.

**Zero eviction authority.** A single enforcement service is, per the design doc, "the only thing allowed to evict." It refuses hard-excluded people (do-not-contact, current employees). It retires anyone who verifiably started a new job since being added. It rescores everyone on recency and depth. It holds a hard cap with a per-run churn ceiling — no single run may replace more than 20% of the list, so the worst possible week is bounded by arithmetic, not by a prompt's mood. And it never removes an entry a human pinned: a recruiter's flag holds for 90 days no matter what the agent does. The doc's own rationale:

> Arithmetic and eviction stay in tested Ruby — that is what makes "a hand-flag does not silently vanish" a guarantee rather than a prompt instruction.

![Diagram of the authority split between the agent and the enforcement service](/blog/assets/images/agent-proposes-authority-split.svg)
*Judgment on the left, authority on the right. State changes only flow through the service.*{: .caption}

The same split governs the agent's memory. Its operating heuristics live in an append-only calibration log: one row per revision, read at the start of each run, revised at the end. Nothing is ever updated or deleted. A human corrects the agent by appending a newer revision; a revert is appending a copy of an older body. A null author means the agent wrote it, a set author means a human did — which doubles as the provenance the learning loop reads. The agent may revise its notes; it may never touch its own prompt. Veto by append.

There's a free lunch hiding in the schema, too: the entries table is the training set. A manual add is a positive exemplar. A human removal is the sharpest negative. A cap eviction is deliberately treated as a weak signal — a system that evicts by its own score and then learns from the eviction is grading its own homework.

None of this holds if the enforcement code is wrong, which is why the session's sharpest bug matters. The agent works through the same public REST API any client would use — the weekly run itself is pure configuration, a cron-triggered routine with a prompt — so the API *is* the perimeter. An adversarial review pass (three independent lenses over the diff, then one judge per finding whose only job was to refute it by reading the code; findings survived only with a concrete failing input) found that the enforcement endpoint accepted a cap override via query param and coerced it with `to_i`. So `?cap=` — a blank string, one flaky client away — became cap 0 and mass-evicted a churn-ceiling's worth of the list while returning 200. Ten of eleven findings survived refutation, in code that had already been reviewed and live-verified by hand. A guarantee with a coercion bug is just a prompt instruction with extra steps.

What the split buys you is a set of statements about the agent that stay true no matter what the model does. "The agent cannot violate a pin" is a theorem about tested Ruby, not a hope about tokens. I wrote this one up because the usual framing — constrain the model — quietly concedes the interesting ground. You don't need a cautious agent if the blast radius is owned by code.
