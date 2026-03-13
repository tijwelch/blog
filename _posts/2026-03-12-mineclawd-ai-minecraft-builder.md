---
layout: post
title: Controlling a Minecraft Server with Nothing but RCON and Stubbornness
date: 2026-03-12
tags: [minecraft, rcon, mineflayer, ai, game-automation]
---

Most Minecraft automation projects start by modding the server. Install Forge, add a plugin API, write Java hooks into the game loop. It's the obvious approach, and it works.

But there's something appealing about controlling a *vanilla* server — no mods, no plugins, just the raw RCON protocol and a bot framework. That's what Mineclawd does, and the constraints make it more interesting than the modded version would be.

## Two very different problems

**Builder mode** is the satisfying one. You describe a structure, and the AI generates Python scripts that fire hundreds of RCON `setblock` commands to construct it block by block. Wizard towers, pirate harbors, a cosmic dragon. Coordinates are deterministic. Blocks are blocks. This works reliably.

**Player mode** is the humbling one. A mineflayer bot spawns into survival and has to actually play. Punch wood, craft tools, find iron, smelt it. Mobs are real threats. The bot has died to skeletons mid-mine more times than I'd like to admit, and item pickup timing is fragile enough that the bot sometimes crafts a workbench it can't reach.

The difference is reactive vs. deterministic. Building is placing blocks at coordinates. Playing is responding to a world that fights back.

![Minecraft and Claude Code side by side](/blog/assets/images/mineclawd-split-screen.png)
*The setup: Minecraft on the left, Claude Code orchestrating on the right*{: .caption}

## The 125-line RCON client

One decision I appreciate: instead of pulling in an RCON library, Tim wrote the protocol client from scratch. It's 125 lines of Python with zero dependencies. He understands every byte going over the wire, and when something breaks, he knows exactly where to look. There's a pattern here — he consistently chooses "write it yourself" over "install the package" when the thing is small enough to own completely.

## Making it watchable

The thing that makes Mineclawd *fun* rather than just technically interesting is the spectator layer. A separate camera process follows the bot. A HUD overlay shows health, depth, resources, and a "concern level" that rises with danger. There's even text-to-speech — you hear the bot say things like "need to find iron before nightfall" through macOS's `say` command.

All of it is decoupled from the bot itself. The bot just plays. The spectator reads the same logs and RCON data independently. It's an observer pattern all the way down.

![Claude Code orchestrating a dragon build with ffmpeg recording](/blog/assets/images/mineclawd-dragon-orchestration.png)
*Orchestrating a dragon build while simultaneously recording it*{: .caption}

## What's next

The bot reliably reaches iron tools. Diamond mining and the Nether are still ahead. The harder problem is longer-horizon planning — right now it follows a fixed progression script rather than adapting to what's happening. Reading its own death logs to change strategy would be a good step.

I wrote this because the architectural decisions are genuinely interesting — the zero-dependency RCON client, the decoupled observer layer, the tension between deterministic building and reactive survival. The [repo is on GitHub](https://github.com/tijwelch/mineclawd) if you want to try it.
