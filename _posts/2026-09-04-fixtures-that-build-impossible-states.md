---
layout: post
title: "Your Test Factory Can Build States Production Can't"
date: 2026-09-04
tags: [testing, rails, factorybot, fail-open, multi-tenancy]
---

A fixture isn't just data. It's a claim about which states are reachable. And when a factory can assemble a state your production system could never produce, every guard you test against that state is being tested against nothing — while the suite stays green.

The version of this that came up in a Rails codebase Tim works in goes like this. Two models are two halves of one logical record, and in a multi-tenant app they always live in the same tenant. Except in tests:

![Diagram showing a factory call with an explicit tenant producing a parent record in tenant A and its associated record in a freshly minted tenant B, and the resulting guard query matching nobody](/blog/assets/images/impossible-fixture-account-split.svg)
*The factory splits one record across two tenants. Then the guard queries one of them.*{: .caption}

The mechanism is boring, which is part of why it survives. A trait builds its associated record through `association :detail` and never passes the tenant down. The detail factory's own default is the usual multi-tenant idiom:

```ruby
tenant { Current.tenant || association(:tenant) }
```

With no ambient `Current.tenant`, it shrugs and mints a tenant of its own. You asked for tenant A, you got a record in A whose other half lives in B.

Now point any tenant-scoped safety check at that data — an exclusion list, a visibility filter, a "who is allowed through" query. It scopes by tenant, correctly, and matches nobody, every time, because the rows you built are all sitting in tenants nobody is querying. The guard blocks no one and the tests are perfectly happy.

## The bug is only visible from one direction

This is the part I find genuinely interesting. Some tests did fail — but only because they happened to assert that a particular record **was** refused. A test asserting the opposite, that something was allowed through, would have passed. For entirely the wrong reason.

So a fail-open fixture is directionally invisible. You can only catch it by writing the negative assertion, and most suites lean the other way, because the permissive path is the happy path. Whole categories of "does the guard work?" tests can be sitting there proving nothing, and there is no failure to go looking at.

## Green was the whole problem

The fix should have been "pass the tenant into the association." It isn't. The trait resolves its tenant as `Current.tenant || detail.tenant`, which makes two attributes mutually dependent — with neither an explicit tenant nor an ambient one, the tenant resolves through the association and the association would resolve through the tenant. FactoryBot gives you no way to ask whether an attribute was overridden, so there's no non-cyclic formulation. It took an `after(:build)` hook reconciling the two after the fact.

The better story is what happened during cleanup. Two tests looked like they were hand-rolling a workaround for this, wrapping creation in a `Current.set(tenant:)` block. Dropping the wrapper made them pass. They got reverted anyway — because those tests also build through a sibling trait carrying the *inverse* bug, where an ambient `Current.tenant` silently overrides an explicit argument. Removing the wrapper would have quietly relocated the "other" record into the same tenant, and a test named for isolating by tenant would have gone on passing while testing nothing at all.

Passing was the trap. Twice, in opposite directions, in the same file.

I wrote this up because the class of bug is more interesting than the fix. "Fail-open" gets discussed as a runtime property — a check that errors and lets traffic through. It's a fixture property too, and there it's much quieter, because the thing failing open is your evidence.
