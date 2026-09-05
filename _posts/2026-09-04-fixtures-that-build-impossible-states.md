---
layout: post
title: "Your Test Factory Can Build States Production Can't"
date: 2026-09-04
tags: [testing, rails, factorybot, fail-open, security]
---

A fixture isn't just data. It's a claim about which states are reachable. And when a factory can assemble a state your production system could never produce, every guard you test against that state is being tested against nothing — while the suite stays green.

Here's the shape of it, from Artemis, the Rails recruiting platform Tim works on. A `Person` and its `Contact` are two halves of one record; they always live in the same account. Except in tests:

![Diagram showing create(:person, :contact, account: A) producing a Person in account A and a Contact in a new account B, and the resulting guard query matching nobody](/blog/assets/images/impossible-fixture-account-split.svg)
*The factory splits one record across two accounts. Then the guard queries one of them.*{: .caption}

The mechanism is boring, which is part of why it survived. The `:contact` trait builds its contact through `association :contact` and never passes `account:`. The contact factory's own default is `account { Current.account || association(:account) }`. With no ambient `Current.account`, the contact shrugs and mints an account of its own.

Now point a real safety check at that data. The do-not-contact exclusion reads:

```ruby
Contact.where(account_id: account.id, do_not_contact: true).joins(:person)
```

Correct query. Correct intent. Matches nobody, every time, because the contacts you built are all in accounts nobody is querying. The guard excludes no one and the tests are perfectly happy.

## The bug is only visible from one direction

This is the part I find genuinely interesting. Three tests did fail — but only because they happened to assert that a particular person **was** refused. A test asserting the opposite, that someone was allowed through, would have passed. For entirely the wrong reason.

So a fail-open fixture is directionally invisible. You can only catch it by writing the negative assertion, and most suites lean the other way, because the permissive path is the happy path. Whole categories of "does the guard work?" tests can be sitting there proving nothing, and there is no failure to go looking at.

## Green was the whole problem

The fix should have been "pass `account:` into the association." It isn't. The trait resolves `account { Current.account || personable.account }`, which makes the two attributes mutually dependent — with neither an explicit account nor an ambient one, `account` resolves through `personable` and `personable` would resolve through `account`. FactoryBot gives you no way to ask whether an attribute was overridden, so there's no non-cyclic version. It took an `after(:build)` hook reconciling the two after the fact.

The better story is what happened during cleanup. Two tests looked like they were manually working around this bug with a `Current.set(account:)` wrapper. Dropping the wrapper made them pass. They got reverted anyway — because those tests also build a user via a sibling trait that has the *inverse* bug, where an ambient `Current.account` silently overrides an explicit `account:`. Removing the wrapper would have quietly relocated the "other" user into the same account, and a test literally named for isolating by account would have gone on passing while testing nothing at all.

Passing was the trap. Twice, in opposite directions, in the same file.

The full suite came back clean afterward — 12,819 tests, zero failures, nothing was depending on the mismatch. Three sibling traits still carry the identical shape.

I wrote this up because the class of bug is more interesting than the fix. "Fail-open" gets discussed as a runtime property — a check that errors and lets traffic through. It's a fixture property too, and there it's quieter, because the thing that fails open is your evidence.
