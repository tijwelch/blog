---
layout: post
title: "The Eval Froze the Part That Mattered"
date: 2026-10-27
tags: [evals, agents, testing, provenance]
---

Every cheap eval of a multi-step agent freezes something. You replay a recorded plan instead of regenerating it, you skip the expensive judging step, you pin the inputs to a date. That's how you get a run that costs nothing and finishes in three minutes instead of three hours and sixty dollars. The catch is that whatever you freeze can no longer move the score, and nothing on the scoreboard tells you which parts those are.

This came up while Tim was building an evaluation for a candidate-sourcing agent. The agent works in stages: a plan step where the model researches a role and writes search strategies and queries, a sweep that runs those queries and pulls people into a pool, and a judge that scores and ranks the pool. The eval grades the result against what recruiters actually did: of the people they approved for a closed search, how many ended up in the pool?

A week of runs had produced a baseline, a fix branch, and a table of numbers. Then Tim asked the question every eval owner should ask on a schedule: "i'm still not really sure what agent code is actually running and being tested."

The honest answer was smaller than the table suggested. A "run" was a local script that loaded a recorded plan from one earlier session, executed only the sweep scripts from a chosen checkout, and posted the pool. No model was involved. The plan was frozen and judging was off. So the eval was a careful, repeatable test of one stage out of three.

That would be fine if the misses lived in that stage. They didn't. On one test search, 49 of 54 missed people traced back to the plan: nobody wrote a query that could have reached them. The eval was most precise about the stage that mattered least, and blind to the one most likely to raise the score.

![Three stages left to right: Plan, frozen; Sweep, live; Judge and rank, off. Below Plan, a red callout reads 49 of 54 misses start here. Below Sweep: the only stage a change can move.](/blog/assets/images/the-eval-froze-the-part-that-mattered.svg)
*The replay was honest. It just could only grade the middle box.*{: .caption}

The numbers made the point on their own. The baseline found 11% of approved people, and three runs of identical code swung by about 1.6 points. Tim's fix branch, a real improvement that pulled 436 more people from past searches, came in at 11.9%. That's +0.8 inside a ±1.6 swing: noise. The fix wasn't useless. It was graded by an instrument that couldn't see where the gains were.

Two changes came out of it, and I think both generalize.

**Every run says what ran.** The app now shows a "What ran" panel on every run: the branch and commit, the scripts executed, whether the plan was frozen, regenerated, or absent, and whether judging was on. The branch table carries a one-line version, like "plan frozen · judging off". Building it turned up a small bonus: the app had been silently dropping the script list the replay was sending all along. So "which code produced this number" wasn't just unlabeled. It was being discarded.

**The frozen stage becomes data you can swap.** Part of the gap between the replay and a full run was that the recorded plan was partly prose. Broad sweeps had been described in words, and words can't be replayed. The plan for the real agent now writes every query as structured data. A frozen plan stops being a limitation of the harness and becomes a switch: freeze it to test sweep changes against a stable target, regenerate it to test plan changes, and the label says which one you're looking at.

The general version: an eval number without its recipe is a rumor. "11%" means nothing until you know which stages were live. The cheapest fix is a label, and it's worth adding before the second branch shows up, not after.

I wrote this up because the most expensive bug in an eval isn't a wrong number. It's a correct number for a question nobody was asking, read as the answer to the one they were.
