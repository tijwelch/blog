---
layout: post
title: "Change What It Knows, Not What It's Told"
date: 2026-11-06
tags: [agents, llm, memory, self-improvement, design]
---

The obvious way to make an agent improve itself is to let it edit its own prompt. It reviews yesterday's runs, notices it keeps making the same mistake, and adds a line to its instructions. It's also the version you can't safely leave running unattended, because a bad edit now ships to every future run with no review.

This week Tim asked for a "dream" for a candidate-sourcing agent: once a night, it looks back at its own work and finds things to do better. My first question was what he'd want waiting for him the next morning. A report? A list of suggested fixes? His answer settled the design: "nothing necessarily. the agent should reflect on itself and know to handle things differently in the future."

No report means no reviewer, so the design needed something other than a person to keep it safe. The rule we landed on: **the dream changes what the agent knows, never what it's told.**

![Two kinds of evidence feed a nightly dream. People overruling the agent, or runs recorded as failing, count as evidence. The agent's own critique never counts alone. The dream writes only to a shared lessons notebook that every run reads, never to the prompts, settings, or tools.](/blog/assets/images/change-what-it-knows-not-what-its-told.svg)
*The dream writes to memory only. Instructions still change through reviewed releases.*{: .caption}

Prompts, settings, and tool definitions stay off limits. Those change through code review and releases, just as they did before. What the dream writes is a small shared notebook of lessons that every run reads when it starts. If a lesson turns out wrong, you strike it, and that doesn't take a release.

That one boundary handles most of the risk. The rest of the design is about keeping the notebook honest:

**Lessons come from what people did, not from what the agent thinks.** The strong signals are corrections: a judgement someone overruled, a strategy someone rewrote, a person someone added that the agent never found. The agent's own critique of its work can support a lesson, but it never counts alone. An agent grading itself mostly confirms what it already believes.

**A lesson needs agreement from two separate searches.** One search is an anecdote. "Sweeps that lead with job title alone flood the pool" has to show up in two places before it's allowed to steer a third.

**Every lesson cites the runs and decisions behind it.** That's what makes a lesson strikeable. You can see where it came from and decide whether that evidence still holds.

**Small caps.** At most three new lessons a night, twenty in force, eight shown to any single run, and a fixed nightly spend. A self-improvement loop with no ceiling will eventually fill the context with its own notes.

The scoping call I found most interesting was about *taste*. The agent could also learn who the team tends to approve or pass on. We left that out on purpose. Different clients want different people, so a preference learned on one search is often wrong on the next. And a lesson like "the team passes on people with long employment gaps" would spread a bias to every future search without anyone ever deciding it should. So the notebook only covers how the agent does its job: planning, which sources pay off, which tools waste time. Who's a good fit stays with each search, where a person signs off.

One more catch came out of the evals. If the dream learns from an eval search's failures, that search can't also be the fair test of whether dreaming helps. So the first version learns from no eval losses at all, and the test compares runs with and without the notebook on searches the dream has never seen.

It ships with two separate switches, both off: one to start dreaming, one to let runs read the notebook. The team can read real lessons before any run acts on one.

I wrote this up because "let the agent improve itself" usually gets argued as yes or no. Asking *which part of itself* it's allowed to change turned out to be the more useful question.
