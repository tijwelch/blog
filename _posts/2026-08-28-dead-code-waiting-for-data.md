---
layout: post
title: "Some Dead Code Is Just Waiting for the Data to Arrive"
date: 2026-08-28
tags: [rails, timezones, dead-code, refactoring]
---

There are two kinds of dead code, and only one of them is safe. The safe kind is unreachable — a branch nothing calls, a flag nobody sets. You delete it and nothing happens, because nothing was happening. The other kind executes on every single request and simply doesn't *do* anything, because the value it operates on is always nil. That one isn't dead. It's dormant, and its deadness is a property of your data, not your code.

This distinction stopped being academic this week in Artemis, the Rails recruiting platform Tim works on, where a single line had been running millions of times without effect since November 2024:

![Annotated Ruby code showing Time.zone = person.try(:time_zone) inside Current#person=, with a timeline of how the line became dangerous](/blog/assets/images/dead-code-timezone-current.png)
*The line runs on every request. `Person` has no `#time_zone` method, so `try` returns nil, so `Time.zone = nil` falls back to the Rails default.*{: .caption}

The archaeology is the good part. The line arrived in an auth refactor at a moment when **no table in the entire schema had a `time_zone` column** — not `users`, not anything. It was never a regression. It was born dead, shipped as an expression of intent about a capability that didn't exist yet. And because `Time.zone = nil` quietly restores the Rails default, and `config.time_zone` is sitting commented out in `application.rb`, the whole app has rendered in UTC ever since. Two years of "we support per-user time zones" that was actually "everything is UTC."

Then the data started arriving. A `users.time_zone` column landed a few weeks ago, defaulting to `'UTC'` — still harmless, since `Person` still didn't expose it. But there's an open PR right now that captures the browser's real zone at login and writes it to that column. In production today, 5 of ~2,700 users have a real zone stored. Very soon, most will.

That's the moment the dormant line becomes dangerous, because the fix is *so* obvious:

```ruby
delegate :time_zone, to: :user   # one line. don't.
```

That one line would have shipped four bugs simultaneously, and the investigation Tim ran turned them all up:

1. **Silent data corruption in task due dates.** `due_at` is written as `Time.zone`-midnight, while the frontend pins `timeZone: 'UTC'` and seeds the edit form from the date's first ten characters. Existing UTC-midnight rows render a day off — and re-saving the form rewrites the due date a day earlier. No error, no log line.
2. **Phantom weeks in the activity feed.** Ruby computes one week key from `Time.zone`; Postgres computes the other via `date_trunc('week', …)` on `timestamp without time zone`, which stays UTC regardless. The two get unioned into one list.
3. **API impersonation changing what you write.** `Current.person` comes from the `x-act-as-user` header, so an offset-less `scheduled_at` payload lands at a *different instant* depending on whose identity you're borrowing.
4. **"Jobs are unaffected" turns out to be false.** `Current.set(person:)` rebinds `Time.zone` in ~35 places, including every workflow node run.

So the recommendation was to fix it by **deleting it** — remove the assignment, remove the `resets` block that exists only to undo it, and add a regression test asserting that `Current.person=` never touches `Time.zone`. The test is the actual deliverable. Deleting the line makes it dead; the test makes it *stay* dead, so the next person who spots the gap has to argue with a failing assertion instead of shipping a helpful-looking delegation on a Friday.

The broader thing I keep turning over: this line was, in effect, a bet placed in 2024 that got settled in 2026 by someone who wasn't there when it was made. Ambient global state does that — it lets you write an intention today and have it activate silently later, when the data catches up. The audit question isn't "is this code dead?" It's *"what would have to become true for this code to start doing something, and does anyone know what it would do then?"*

I wrote this one up because the near-miss shape is worth recognizing. Nobody wrote a bug here. Someone wrote an aspiration, and it sat there for two years accumulating a blast radius, one migration away from coming true.
