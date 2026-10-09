---
layout: post
title: "Provision From State, Not From the Event"
date: 2026-10-09
tags: [architecture, reconciliation, callbacks, rails, scheduling]
---

A lot of provisioning code is a list of triggers. When a user is created, make their bucket. When a recruiter is made owner of a search, create their booking page. Each trigger is fine on its own. The bug is the list, because the list is a hand-written guess at every path that leads to "this resource should exist", and the paths you didn't guess produce nothing. No error, no retry. Just a thing that should exist and doesn't.

Tim hit two of these in one week, in the same scheduling feature, and both got the same fix: stop acting on the event, converge on the state.

The first one. Every search has a booking page on its host's calendar, created by a callback that fires when a person gains the owner role. The job also refuses to run unless the new owner has a connected calendar, which is sensible. Now watch what production did. Person A is the first one added to an empty team, so the app makes them owner and builds a page on their calendar. Person B is promoted to owner before connecting a calendar, so the job no-ops. Person A is demoted. Nothing listens for that, because the condition reads "do the new roles include owner", a question about gaining the role, not losing it. The search's shareable link falls back to the only page it has, A's, next to copy that says to paste this link into mass outreach. Someone did, into six email templates. For two weeks, screening calls for B's search landed on A's calendar, and the host-selection logic, the first suspect, was never involved.

![Two columns. Trigger list: B promoted (no-ops, no calendar yet), B connects a calendar (nothing listens), A demoted (nothing listens), so A's page is the only one and the link points at it. Converge on state: all three events feed one reconcile step that computes the desired host from state and makes the pages match.](/blog/assets/images/provision-from-state-not-from-the-event.svg)
*Same three events. The left side has a trigger list; the right side has a rule.*{: .caption}

The second one is smaller and has the same shape. After a candidate books, a job rewrites the calendar event with the candidate's name and details. It ran once, on booking. When a candidate reschedules, the provider rebuilds the event from its template and the rewrite is gone. About forty percent of recent events had reverted to the bare template title, and nobody had tied it to rescheduling, because the first booking always looked right.

The fix starts by writing down the rule instead of the triggers. Who should host this search? The default meeting host if they're on the team with a connected calendar, otherwise the owner:

```ruby
def scheduling_host
  default_host = job_detail&.scheduling_host
  return default_host if default_host && EligibleHosts.call(bucket:).include?(default_host)

  scheduling_owner
end
```

Then the provisioning job becomes idempotent: compute the desired host, create their page if it's missing, retire the pages that no longer match. Once the job is safe to run any number of times, the trigger list stops being a correctness problem and becomes a latency problem. You still wire it to the events you know about (owner changes, default host changes, a calendar grant turning valid), but a missed event now means "fixed on the next one" instead of "wrong until someone notices". A one-off task sweeps the searches that drifted before the fix shipped, which is the same job run against every search instead of one.

The reschedule bug got the same treatment with less work, because the rewrite job was already idempotent: it writes its block behind a marker and replaces everything below the marker on a rerun. All it needed was a second caller, the reschedule webhook.

Kubernetes people have a name for this: edge-triggered versus level-triggered. A controller looks at the current state and the desired state and closes the gap, rather than reacting to "pod created". The reason is exactly the failure above. Edges get missed. A watch disconnects, a message drops, a precondition arrives after the trigger. Level-triggered code doesn't care which event woke it up, because it recomputes from scratch every time.

What strikes me is how natural the trigger list looks in Rails. `after_commit :provision, on: :create` reads like the obvious design. It is also a list of edges with the level rule written nowhere, so the rule lives only in the heads of whoever wrote the callbacks. I wrote this up because the first investigation went straight to the host-picking logic, which was correct, while the real bug was a callback condition that only ever asked about one direction of a role change. If the desired state had been a function, "why doesn't the page match it" would have had somewhere to start.
