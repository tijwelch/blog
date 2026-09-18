---
layout: post
title: "Eventually Consistent Was a Setting"
date: 2026-09-18
tags: [opensearch, search, architecture, rails, consistency]
---

There's a rule most of us absorbed without ever deciding to believe it: don't read a search index right after you write to it. The index is eventually consistent, the write lands a second or two later, and the user sees stale data. So you keep a second read path against the database for anything freshly mutated, and you pay for that second path forever.

That rule got tested this week, and it turns out it's usually not a property of the index. It's a scheduling default.

The setup: a curated list of people needs the same filter panel the main people-search page already has — seven sections of title, company, location, industry, the works. Those filters run against OpenSearch. But whether someone is *on the list* lives in Postgres, alongside who pinned them, why, and when. Two obvious designs. Either Postgres decides who's on the list and hands OpenSearch a set of ids to filter within, or the list membership gets projected into the search document and one query does everything.

The second one is clearly nicer — you get the engine's sort and cursor paging for free instead of reimplementing them. The objection to it was freshness, and it was concrete: every action on that page (pin, remove, add back) commits and redirects into a roster refetch that lands about half a second later. An async partial reindex lands one to two and a half seconds after commit, with a twenty-to-sixty-second tail under queue backlog. You'd click Remove and the person would still be sitting there.

That's a real measurement, and it killed the design — until someone pointed out the measurement was of a default, not a constraint. The index is a projection of Postgres. If a surface needs to read its own write, reindex synchronously.

```ruby
# The page reads the roster back from the index right after the redirect,
# so the projection is updated inline and made searchable before we return.
def reindex_person
  person.reindex(:search_data_hot_list, mode: :inline, refresh: true)
end
```

That's the whole reversal. `mode: :inline` skips the job queue; `refresh: true` makes OpenSearch flush the doc into the searchable segment before the call returns. Two keywords on an `after_commit`, scoped to one bucket of fields so it's a small partial update rather than rebuilding a person's whole document. Everything downstream — filters, sort, paging — collapses into one ordinary search query.

![Three write paths on a shared timeline: async lands too late, Postgres-owns-ids is fresh but loses native sort, inline plus refresh is fresh and still one query](/blog/assets/images/eventually-consistent-was-a-setting.svg)
*The same click, three scheduling choices.*{: .caption}

The interesting move isn't the keywords, it's the reframing. "Is this index eventually consistent" is the wrong question, because the answer is a per-write-path decision, not a property of the system. The right question is: which writes does a user read back within a second? That's a tiny set — hand-curation actions, almost never bulk jobs — and you can pay synchronous cost on exactly those while everything else stays async.

Two things I'd have gotten wrong on my own, both now load-bearing comments in the code:

**A partial reindex is a merge, not a replace.** If your `search_data` method omits a key because the value is absent, the old value survives in the document. So every key gets emitted on every write, `nil` included. The bug this prevents is the worst kind — a field that's correct in the database and confidently wrong in search, with nothing failing anywhere.

**Never index a time-dependent predicate as a boolean.** "Is this person pinned" is true today and false on Tuesday, with no write in between to trigger a reindex. So the document stores `pinned_until` as a date, and the query is `{ range: { 'hot_list.pinned_until' => { gte: 'now' } } }`. An indexed value is a claim made at write time; anything that can quietly become false is a lie waiting to be served.

And one fact stayed out of the index entirely: whether someone is about to age off the list, which is computed per request from several other tables. That's the real boundary — facts get projected, judgments get computed at read time.

I wrote this up because the first analysis was thorough, well-measured, and landed on the wrong answer. It treated an observed latency as a law of the system. The most expensive assumptions are the ones you can cite a number for.
