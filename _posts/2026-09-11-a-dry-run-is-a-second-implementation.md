---
layout: post
title: "A Dry Run Is a Second Implementation of the Rule"
date: 2026-09-11
tags: [rails, agents, dry-run, testing, design]
---

Any UI that tells you what an automation is *about to* do has quietly forked your rule into two implementations. One that decides, and one that predicts. They start out agreeing, because you wrote them an hour apart. Nothing keeps them agreeing.

This came up while Tim was building a page for a list that a weekly agent prunes on its own. The list is curated — some people are on it because an agent proposed them, some because a teammate added them by hand — and every Monday a service walks it and removes whoever no longer belongs. The page wanted to be kind about that: show a warning tag on anyone the next run would drop, and give you a Keep button to save them first.

That's a good feature. It's also a second implementation of the pruning rule, and the ways it can be wrong are nastier than ordinary bugs, because a preview's entire job is to be believed.

The rule has ordering in it. Hard exclusions — do-not-contact, teammates — outrank a pin: a human flag is a judgment call about fit, and those facts aren't about fit. Retiring someone who started a new job does *not* outrank a pin. So the preview has to reproduce not just two predicates but their precedence:

```ruby
def call
  refused  = refusals
  retiring = retirements(
    @entries.reject { |e| refused.key?(e.id) || e.pinned? }
  )
  refused.merge(retiring)
end
```

Get that backwards and you ship a Keep button that doesn't keep — it pins someone whose removal ignores pins, and Monday takes them anyway. A wrong preview is worse than no preview, because it spends trust it can't repay.

![Enforce and Departures side by side: three removal paths, two restated in prose, one missing, with a single shared resolver underneath](/blog/assets/images/dry-run-second-implementation.svg)
*Solid line: one function. Dashed line: two paragraphs of prose that happen to agree today.*{: .caption}

What's interesting is which part got shared. Not the predicates — the *resolver*. Mapping a person to their employment record is polymorphic and fiddly, two different owner types with two different join paths, and it was the piece most likely to drift into two subtly different answers. That got pulled out into one function both sides call:

```ruby
profile_ids = EmploymentProfiles.ids_for(entries.map(&:person))
```

The predicates stayed duplicated. And honestly, that's a reasonable trade — inverting a service so a live code path can run it in "tell me but don't do it" mode is a real cost, and one shared function bought most of the safety.

But look at the third row in that diagram. The enforcer has a removal path the preview doesn't model at all: evicting the lowest-scoring entries when the list is over a size cap. The preview is complete right now *only because* today's caller doesn't pass a cap — it's an optional argument on the API endpoint. The preview isn't correct by construction. It's correct by a value someone didn't send.

The tests don't catch this, and they can't, because they assert the preview matches the rule as restated *in the test*. Six cases, all green, none of them an opinion about the enforcer. The test that would actually hold is differential: run both over the same fixtures, assert the predicted set equals the set that enforcement actually removes. That test fails loudly the day someone adds a fourth removal reason, which is exactly the day you want to hear about it.

This is the whole reason people trust `terraform plan`: plan and apply aren't two implementations that agree, they're one code path run twice. Most dry runs aren't built that way. Most dry runs are a second opinion wearing the first one's clothes.

I picked this out of the week's sessions because the bug here hasn't happened yet, and the interesting ones rarely get written up before they do.
