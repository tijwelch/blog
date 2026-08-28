---
layout: post
title: "A Model Asked to Diagnose Will Always Return a Diagnosis"
date: 2026-08-28
tags: [llm, github-api, dashboards, agents, ground-truth]
---

GitHub will tell you, with a straight face, that a branch doesn't require signed commits while refusing to merge your unsigned commits into it. The legacy branch-protection endpoint answers one question:

```bash
gh api repos/OWNER/REPO/branches/main/protection --jq .required_signatures.enabled
# false
```

The ruleset covering the same branch answers a different one, and it's the one that's actually enforced:

```bash
gh api repos/OWNER/REPO/rules/branches/main --jq '.[].type'
# ... required_signatures
```

Two APIs, same branch, opposite answers. Worse, there is no PR-level field that reports it at all. So a pull request sits there mergeable, CI green, `mergeStateStatus: BLOCKED`, with nothing anywhere saying *why*.

That gap is where this gets interesting. Tim has a little native macOS window that shows what's in flight across his projects — PRs, branches, tickets — and the prose in it is written by an LLM on a refresh loop. Given a PR that's green and blocked and silent about the reason, the model filled the hole. Twice, on separate runs, it wrote **"needs a reviewer"** for PRs that were actually blocked on signatures.

That's not a hallucination in the lurid sense. It's the single most probable completion. Green CI plus blocked plus no stated cause really does mean "waiting on review" the overwhelming majority of the time. The model produced the likeliest answer, which happened to be the wrong one — and the cost of a plausible wrong answer here isn't a laugh, it's going and pinging a colleague about a PR they were never holding.

The fix wasn't a better prompt. It was taking the fact away from the model. The refresh script now computes the blocker in bash, as an explicit precedence chain, before the LLM sees anything:

```bash
blocker: (if .mergeable=="CONFLICTING" or .mergeStateStatus=="DIRTY" then "merge conflicts"
  elif (ci=="red")     then "failing CI"
  elif (ci=="pending") then "CI still running"
  elif ($u > 0) and .mergeStateStatus=="BLOCKED"
                       then "\($u) unsigned commits (main requires signed)"
  elif .reviewDecision=="CHANGES_REQUESTED" then "requested changes"
  elif .mergeStateStatus=="BLOCKED"         then "branch protection"
  else null end)
```

`$u` is the part nothing hands you: a per-PR count from walking `pulls/N/commits` and filtering on `.commit.verification.verified`, because no summary field exposes it. The prompt's job shrank to one instruction — quote `blocker` verbatim.

![The In Flight window showing a PR blocked on unsigned commits](/blog/assets/images/blocker-diagnosis-in-flight.png)
*The same fact, computed rather than inferred: "CI-green but blocked by 19 unsigned commits."*{: .caption}

The general shape: **the model writes the words, the script writes the state.** Anything with a computable ground truth should be computed, and the LLM's remaining job is phrasing. It sounds obvious until you notice how much of a generated status surface is quietly in the other category — a progress bar, a severity, a "waiting on X." In the same redesign the progress bar got deleted outright, on the grounds that it had no honest source. Better no bar than a fake one.

What makes this failure mode nasty is that a model will not spontaneously return *unknown*. It has no mechanism for it unless "unknown" is a value your data can actually hold. Ask for a diagnosis and you get a diagnosis, every time, at full confidence, and a wrong status renders identically to a right one.

I wrote this one up because the bug is invisible by construction. Nobody audits a dashboard that reads fine.
