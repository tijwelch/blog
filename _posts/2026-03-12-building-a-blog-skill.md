---
layout: post
title: I Built a "Post This to the Blog" Button for Claude Code
date: 2026-03-12
tags: [claude-code, skills, developer-tools, automation]
---

I have a documentation problem. I build cool stuff all the time — features, tools, integrations — and then none of it gets written up. The bottleneck isn't ideas or even writing ability. It's the activation energy of switching from building mode to writing mode.

So I built a Claude Code skill that lets me say "post this to the blog" at the end of any conversation, and it writes up what happened as a blog post. You're reading the first one.

## The idea

Claude Code has a skill system — markdown files that act as reusable prompts with trigger phrases. I already have skills for things like telling jokes, reviewing PRs, and playing Minecraft. Why not one that turns a conversation into a blog post?

The whole thing is two pieces:

1. A `~/blog/` folder for markdown files
2. A skill file at `~/.claude/skills/blog-post/skill.md`

That's it. No static site generator, no build step, no deploy pipeline. Just markdown files in a folder. If I ever want to turn them into a site, the content is ready. But the point right now is just to capture things.

## The skill

The skill file is around 50 lines. It tells Claude to:

- Review what happened in the conversation
- Write it up in first person, casual tone
- Save it to `~/blog/YYYY-MM-DD-<slug>.md`
- Keep it short (300-600 words)

The style guide is the most important part. Without it, you get corporate blog voice — "In today's fast-paced development landscape, documentation remains a critical..." No. Just write like a developer talking to other developers.

```yaml
---
name: blog-post
description: Create a blog post from the current conversation. Use when user says
  "post this to the blog", "blog this", "blog post"...
---
```

Trigger phrases include "post this to the blog", "blog this", "document this" — basically anything that sounds like you want to capture what you did.

![Tim's Build Log live on GitHub Pages](/blog/assets/images/blog-skill-live-site.png)
*The blog, live on GitHub Pages — two posts and counting*{: .caption}

## What I like about this

The friction is now near zero. I don't have to context-switch. I don't have to remember what I did. I don't have to write anything. I just say three words at the end of a session and a post appears.

The posts won't be perfect. But they'll exist, which is infinitely better than the nothing I was producing before.

## What's next

It's live on GitHub Pages now — Jekyll renders the markdown automatically on push. The skill handles the git commit and push too, so the full flow is: say "post this to the blog" → post written → committed → pushed → live. Future me will thank present me when I'm trying to remember how I set something up six months from now.
