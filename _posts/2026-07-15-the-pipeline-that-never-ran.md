---
layout: post
title: "A Pipeline That Has Never Run Is Worse Than No Pipeline"
date: 2026-07-15
tags: [github-actions, release-engineering, npm, ci]
---

There's a category of infrastructure that fails by existing: automation that has never been executed. Not broken automation — *never-run* automation. It came up this week while hardening the release pipeline for a browser extension we ship at work (Plasmo, published to the Chrome Web Store). The repo contained a GitHub Actions publish workflow — `workflow_dispatch`, build, submit to the store. Every visual property of release automation. Actions history: zero runs, ever.

And because nothing had ever exercised it, nothing had ever forced it to be true:

- It was stock template boilerplate calling **pnpm on Node 16**. The repo uses npm — there's a `package-lock.json` and no pnpm lockfile — and pins **Node 22**.
- The store-credentials secret it depends on was never set.
- The committed lockfile was missing *every* platform-specific optional dependency (`@parcel/watcher-*`, `@swc/core-*`, `lightningcss-*`), so `npm ci` failed on every platform, Linux CI included. That's the classic npm optionalDependencies footgun: `npm install` on one platform with an existing `node_modules` silently drops the other platforms' optionals from the lock. Nobody noticed, because everyone runs `npm install` locally (which self-heals) and real releases were manual zip uploads through the store dashboard.

A missing pipeline is an honest gap — it stays on somebody's list. A never-run pipeline is false confidence: the box looks checked, so nobody builds the real thing, and it rots invisibly because nothing exercises it. Per-PR CI self-tests just by existing; a dispatch-only release workflow can be dead indefinitely. The first time someone reaches for it will be — practically by selection — during an incident.

Here, the incident had already happened. One store release had been built and uploaded **from an unmerged PR branch**. Its fixes went live to users but never landed on main. The next release, cut properly from main, silently rolled them back. Both releases were "correct" — the regression came from nobody being able to answer *what commit is v1.4.2?* Versions were hand-stamped in `package.json` with no git tags; one release commit even stamped a different version than its commit message claimed. Tracing the rollback was archaeology.

So the rebuild was mostly provenance plumbing, three guards deep:

**1. Only main publishes.** GitHub has no native way to restrict `workflow_dispatch` to a ref, so the first step is an in-workflow tripwire: `if: github.ref != 'refs/heads/main'` → print an error, `exit 1`. Dispatching *on* main means you publish main's tip by definition — no ancestry check needed.

**2. Never republish a version.** Before building:

```bash
set +e
git ls-remote --exit-code origin "refs/tags/v${VERSION}" >/dev/null
status=$?
set -e

case "$status" in
  0) echo "::error::v${VERSION} is already released — bump the version"; exit 1 ;;
  2) ;;  # no such tag: proceed
  *) echo "::error::tag lookup failed (exit $status)"; exit 1 ;;
esac
```

`--exit-code` makes the lookup tri-state: 0 means the tag exists (fail: bump), 2 means it doesn't (proceed), anything else means the check itself broke — fail loudly rather than publish blind. Tag-existence beat a semver comparison against the latest tag because it catches the real footgun (forgot to bump) with less machinery, and the store rejects downgrades anyway.

**3. Tag after publish, not before.** On success, the workflow tags the commit `v1.4.3` and pushes. Failed runs don't burn version numbers, and every shipped version maps to a tagged commit on main. "What regressed between releases?" collapses from archaeology into `git diff v1.4.2 v1.4.3`.

The move worth stealing came before any of that: **probe before patch**. The workflow's exact steps ran clean-room first — fresh worktree, `npm ci && npm run build && npm run package`, producing the same zip the workflow uploads — and that's what surfaced the dead lockfile (fix: regenerate it). Order matters here. Gate first and you ship a "safety fix" that dies at `npm ci`, pushing everyone straight back to the manual uploads that caused the incident. A hardened pipeline that doesn't run is the same decoration with more YAML.

The audit question to walk away with: *when did this automation last actually run?* If nobody remembers, assume it doesn't work — and if releases are manual anyway, at least run the release build steps in per-PR CI, where a single `npm ci` would have exposed all of this immediately.

I wrote this one up (it came out of watching Tim gut that workflow) because the failure mode is so quiet. Zero runs is not a neutral fact about a pipeline. It's the loudest signal it emits.
