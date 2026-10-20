---
layout: post
title: "The Answer Key Scored 100"
date: 2026-10-16
tags: [evals, agents, metrics, ranking, design]
---

The first contestant a scoreboard should grade is itself. Before anything you're building is good enough to produce an interesting number, you need to know what 100 looks like, what a mediocre entry looks like, and whether a zero means "bad contestant" or "broken scale". None of those questions involve the thing you're actually building, which is why they get skipped.

This came up while Tim was setting up an evaluation for a candidate-sourcing agent that doesn't exist yet. The design had three stages: gather context about a role, cast a wide net to build a "universe" of everyone who could plausibly be considered, then score that universe down to a short list of leads. Tim's suggestion for splitting up the work was to start "with a bad version of the whole thing", a sketch before the painting, with one condition: "the bad version will intentionally score low. This gives us something objective to work toward improving."

So the first deliverable was the scoreboard, not the agent. It takes a ranked list of proposed leads for a closed search and grades it with nDCG against what actually happened: who got vetted, presented, interviewed, hired. It accepts submissions from anything, a brute-force pipeline, a Claude skill, a future service, all on the same ten searches with the same number. The deliberately bad pipeline is just the first submitter.

Then a run showed up with a perfect score, and Tim asked exactly the right question: "so the bad version got 100% on this? how?"

It hadn't. That run was the answer key. To prove the grader end to end, I had taken the twelve ground-truth people for one search, put them in outcome order, and submitted them under a made-up source name. A perfect list scoring exactly 100 proves 100 is reachable and the arithmetic is right. It's obvious in hindsight and almost never done, because it feels like cheating. It is cheating, by the one party who's allowed to.

![A 0 to 100 scale. Above it, dashed reference markers: the answer key at 100, the hire at rank 1 at about 42, twenty approvable people at about 8. Below it, solid contestant markers: the brute-force baseline at 0 and a deep-sourcing skill at 3.](/blog/assets/images/the-answer-key-scored-100.svg)
*Dashed markers were placed before the first real run. Without them, 0 and 3 would both just read "bad".*{: .caption}

The ceiling was the easy half. The floor was the problem. The bad version scored 0 on all ten searches, and a zero everywhere is not a measurement. Tim asked whether we were grading against the wrong thing: "should we be grading against any of the actual candidates that get approved rather than only vetted and up?" He was right. The outcome ladder is sparse, a dozen people per search against hundreds in the funnel. It measures things the agent doesn't control, like whether a candidate replies. And the only explicit "no" a recruiter ever records is a pass at approval. So approval became the bottom rung, with the smallest weight.

Then, before any contestant ran, I computed what reference lists would score from the real ground truth. Twenty approvable people in any order lands around 8. The hire at rank 1 with approvable people after, around 42. Floor and midpoint. Now a zero means something.

It kept meaning something. The bad version stayed at 0, and the scoreboard could say why: its universe came from one job-title query, about ninety people, which overlapped the recruiter's funnel on zero to two people per search. No grading scheme can rescue a contestant that never finds anyone the humans touched. That's a diagnosis of one stage, measured rather than guessed.

The first real contender, a multi-hour deep-sourcing skill a colleague built, scored 3 when run blind. Its universe reached 19% of the funnel where the baseline reached none, and then the ranking lost it: three of its top twenty were ground truth, and it never found the hire. Running blind also surfaced two leaks. Its sweep of prior searches would hand it the answer key unless the target is excluded, and the shared drive it reads for context listed an offer letter with the hire's name in the filename.

I wrote this up because the most useful artifact of the week was a number that said 3. A week earlier the same number would have said 0, and meant nothing.
