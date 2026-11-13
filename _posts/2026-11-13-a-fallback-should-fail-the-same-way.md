---
layout: post
title: "A Fallback Should Fail the Same Way"
date: 2026-11-13
tags: [networking, dns, debugging, minecraft, reliability]
---

A backup is only a backup if it gives the same answer. If it gives a *different* answer that also looks fine, it isn't protecting you from failure. It's hiding the failure.

This came up while Tim was trying to get a Nintendo Switch onto the Minecraft server running on his Mac. The server is Java Edition and consoles run Bedrock, so a translation proxy (Geyser) sits in front of it. That part worked. The hard part was getting the console to *find* the server, since consoles won't let you type in a server address.

The well-known workaround is a DNS trick. You point the console's DNS at a community server that intercepts the lookup for one of the built-in "featured servers." Tap that tile, and instead of the real server you get a little menu where you can type your own address. The setup guides all say to fill in two fields: the redirect server as primary DNS, and a public resolver like `8.8.8.8` as secondary, so the rest of your internet keeps working if something goes wrong.

Tim set it up, tapped the featured server, and the real one loaded. The full lobby came up and everything looked normal.

The question was what that observation actually told us. There were three things that could happen when he tapped the tile:

- **The redirect menu appears.** The trick works.
- **It can't connect.** The redirect server is reachable but broken.
- **The real server loads.** The console's lookups are being answered *normally*.

The third one is the giveaway. If the redirect server were answering, the real server could never load. Something else was answering. The most likely story was that the primary was dead, the console quietly tried the secondary, and `8.8.8.8` did exactly what a good resolver does: it returned the real address. The fallback turned "the trick is down" into "everything is fine, just not what you wanted," and nothing on screen said a failover had happened.

![Two DNS setups compared across three outcomes. With a public resolver as backup, the real server loading looks normal but means the redirect failed silently. With a second redirect server as backup, the same outcome means the trick itself is broken, and a total outage shows up as a connection error.](/blog/assets/images/a-fallback-should-fail-the-same-way.svg)
*Same outcomes, different meanings. A backup that gives a different answer makes them ambiguous.*{: .caption}

The fix wasn't a better primary. It was a backup that gives the same answer: a second community redirect server as the secondary instead of a public resolver. With that setup, each outcome on the console means only one thing. If both redirect servers are down, you get a connection error, which is loud and honest. And if the real server *still* loads, a single dead server can't be the cause anymore.

That last case is what happened. The real lobby kept loading, so the problem wasn't one flaky community box. The DNS trick as a whole wasn't working for this console. That's a much more useful conclusion than "keep fiddling with DNS," and it sent the setup off in a different direction: a plugin that signs a spare account into the console's online service and advertises the server as a friend's world, so it shows up in the Friends tab with no DNS tricks at all.

The general version: when you add a fallback, ask what it returns, not just whether it returns. A fallback that serves a *different, plausible* answer, like a public resolver behind a redirect, a cached page behind a live one, or a default value behind a failed lookup, removes the signal you'd need to notice the primary is gone. If you can't make the backup give the same answer, make it fail loudly.

I wrote this one up because the diagnosis didn't need logs, packet captures, or a single command. It came from asking which of three outcomes could only happen if something unexpected was answering the question.
