---
layout: post
title: "Translate Before You Prompt: Turning Complex Workflow Trees into One-Sentence Summaries"
date: 2026-03-13
tags: [prompt-engineering, ai-features, structured-prompting, rails]
---

There's a prompting pattern I keep seeing in production AI features that doesn't get enough attention: **translate your data into human language before the AI ever sees it.**

The idea is simple. If your database stores a workflow as a tree of nodes with UUIDs, don't send UUIDs to the model. Resolve them first. Turn `to_stage_id: "01980f5e-2c50"` into `to_stage_name: "Approved"` before the prompt is constructed. The AI works with words, not identifiers — give it words.

This came up in a feature Tim built for SearchOS, a recruitment platform. The product has visual workflow automations attached to hiring pipeline stages — things like "when a candidate enters Approved, check if we have a warm connector, request an intro if so, otherwise send a LinkedIn cold outreach." These workflows are powerful, but they're built as node trees that look like programmer flowcharts. A teammate glancing at a stage column has no idea what the automation actually does without opening the editor and reading every node.

The solution: generate a one-sentence AI summary when the workflow is saved, then show it in a popover.

## The Anatomy of the Prompt

Two parts. A **system prompt** that stays identical for every workflow:

```
You are describing workflows for a recruiting company
that automates candidate outreach.

Describe what the workflow does in ONE sentence.
Start with "When" and focus on the business outcome.
```

And a **user prompt** that changes completely each time — the structured workflow tree, with all IDs pre-resolved to names:

```
trigger: candidate_moved
  - from_stage_name: Leads
  - to_stage_name: Approved
condition: has_connector
  action: request_intro
    - stage_name: Intros Pending
  action: send_linkedin_inmail
    - stage_name: Cold Outreach
```

Out comes: *"When a candidate moves from Leads to Approved, the workflow pauses briefly and then routes them into a warm intro path via a connector if one exists, otherwise it initiates cold outreach via LinkedIn InMail."*

## What Makes It Work

Three things stand out:

**The interpretation guide.** The system prompt includes a dictionary mapping technical patterns to business concepts — "multiple emails in sequence = nurture campaign," "intro accepted = warm outreach path." This bridges the gap between what the data structure says and what a recruiter would understand. Without it, the AI describes mechanics. With it, it describes intent.

**The constraint.** "ONE sentence. Start with When." Tight output constraints prevent the model from doing what it naturally wants — enumerate every step. Good and bad examples in the prompt reinforce this by showing, not telling.

**Pre-computation.** The AI call happens once in a background job when the workflow is saved. When someone clicks the popover, it's just reading a text field from the database. No latency. This is the kind of decision that separates a demo from a feature — users don't wait for AI, they read a cached result.

## The Claude Code Parallel

If you use Claude Code with a `CLAUDE.md` file, you're already doing this exact pattern. Your CLAUDE.md is the system prompt — persistent instructions that shape every response. Your typed question is the user prompt — different every time, same rules. The only difference here is that *the code writes the user prompt automatically* by reading the workflow tree, resolving references, and formatting structured text. No human types anything.

I wrote this up because the "translate before you prompt" pattern is broadly useful and under-discussed. Most tutorials show you how to write better instructions to the AI. Fewer show you how to prepare better *data* for the AI. The data preparation is where the real leverage is.

[Explore the interactive explainer](/blog/assets/images/workflow-summary-explainer.html) — it walks through the full architecture with diagrams.
