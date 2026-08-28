---
layout: post
title: "The Skill Name Is the Router, Not the Description"
date: 2026-08-28
tags: [claude-code, skills, evals, llm, measurement]
---

Every guide to writing a Claude Code skill tells you the same thing: the `description` in the frontmatter is the primary triggering mechanism. Write it carefully. Enumerate the phrases users might say. Make it a little pushy, because models tend to under-trigger. There's even a whole optimization loop dedicated to rewriting descriptions for better routing.

I spent an afternoon this week trying to prove that description quality matters, and failed completely.

The setup was a regression harness for a plugin of ~20 internal skills at Tim's company, where merging to main is an immediate release to everyone. CI checked that skills were *well-formed*. Nothing checked whether an edit made one *worse*. So: run real sessions against two arms of the same cases — the working tree versus the plugin at the merge base — and fail only on a drop against baseline.

Then came the part that actually validates a harness, which is trying to break something on purpose. I degraded one skill's description and expected the suite to go red.

![Three injected description regressions and their results — all still triggered](/blog/assets/images/skill-routing-name-vs-description.png)
*Each arm scored against the merge-base version of the same skill. Haiku, three runs per case.*{: .caption}

Injection one was a plausible tidy-up — the kind of edit someone makes without thinking. Not caught. Injection two broadened the description to claim a sibling skill's territory, which should have tripped the `skill_not_used` guard on the sibling's own test case. It didn't. So I stopped being subtle and rewrote the description to say the skill **formats bibliographic citations in APA style**. Six out of six triggers, on prompts about evaluation rubrics.

At that point the honest thing to suspect is your own plumbing. So I planted a sentinel string in the description and asked the model to quote skill descriptions back at me. It came back verbatim. The edits were reaching the session. I also rewrote the prompts to avoid the skill's own vocabulary — "nail down what we're evaluating against" instead of "rubric" or "scorecard" — in case the wording was doing the matching. No change.

The remaining explanation is the boring one: **the skill's directory name is the router.** A directory called `create-scorecard` gets selected for a scorecard-shaped request whatever its description claims, as long as no sibling competes for that request.

That has a sharp consequence for anyone building eval suites around skills. A trigger-based suite detects exactly three things: a skill removed or renamed, a new skill stealing a request, and over-triggering on a genuinely ambiguous name. It cannot see description drift or boundary blur, which are the regressions people actually ship. Catching those means grading the *artifact* the skill produces, not the routing decision. Selling a green routing suite as a skill health check is selling the wrong instrument.

The authoring corollary is more useful: naming is the high-leverage decision, and a badly-named skill probably can't be rescued by a better description. Spend the thought on the directory name.

Scope caveat, because it matters: measured on Haiku, ~20 skills, no directly competing sibling. Whether a larger catalog or two overlapping names restores description sensitivity is untested.

I wrote this up because the interesting result was the null one. The harness worked fine; the assumption it was built on didn't survive contact with a measurement, and I'd rather that be written down than quietly corrected.
