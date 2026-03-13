---
layout: post
title: "Mineclawd: Teaching Claude to Build (and Survive) in Minecraft"
date: 2026-03-12
tags: [minecraft, claude-code, ai, mineflayer, rcon]
---

I built a system that lets Claude Code control a vanilla Minecraft server in real time. You talk to it, it builds things. Or you tell it to survive, and it punches trees, crafts tools, mines iron, and fights zombies — all on its own. It's called Mineclawd.

## Two modes

**Builder mode** is the flashy one. Claude generates Python scripts that fire hundreds of RCON commands to construct structures block by block. Wizard towers, pirate harbors, a cosmic dragon, a full city — you describe it, Claude builds it while you watch from a good camera angle. It even choreographs cinematic reveals, teleporting you to the right vantage point before the build starts.

**Player mode** is the harder one. A mineflayer bot named "ClaudeBot" spawns into survival and has to actually play the game. No cheats. It punches wood, crafts a crafting table, makes stone tools, finds iron, smelts it. Mobs are real threats — I've watched it get killed by skeletons mid-mine and respawn to try again.

## The stack

The whole thing runs on surprisingly little:

- A **vanilla Minecraft server** (1.21.4, no mods) with RCON enabled
- A **125-line Python RCON client** with zero external dependencies
- A **mineflayer bot** in Node.js for player mode
- **Bash scripts** to glue it all together

The RCON client was one of the better decisions. Instead of depending on a library, I wrote a clean implementation of the protocol from scratch. It's simple, reliable, and I understand every byte going over the wire.

## Making it watchable

The thing that took Mineclawd from "cool demo" to "actually fun" was making it observable. A separate camera process follows the bot around. A HUD overlay shows what it's doing, its health, depth, resource progress, and a concern level that goes up when things get dangerous. There's even text-to-speech for the bot's inner monologue — you hear it say things like "need to find iron before nightfall" through your speakers via the macOS `say` command.

All of these are separate processes reading from shared logs and RCON, not coupled to the bot internals. So the bot just does its thing, and the spectator experience is layered on top.

## What's hard

Getting a bot to reliably play Minecraft is way harder than getting it to build. Building is deterministic — place block at coordinates. Playing is reactive — the world fights back. Item pickup timing is fragile. Furnace smelting requires waiting. The bot sometimes places a crafting table in a spot it can't reach. Deaths mean recovering context about what you were doing and where you were.

The current bot (third major iteration) can reliably get to iron tools. Diamond mining and the Nether are still on the roadmap.

## What's next

The builder mode is solid and genuinely impressive to watch. Player mode needs work on longer-horizon planning — right now it follows a fixed progression script rather than adapting to what's happening. I want the bot to read its own death logs and change strategy, and eventually make decisions through a real-time chat bridge with Claude.

The [repo is on GitHub](https://github.com/tijwelch/mineclawd) if you want to try it. All you need is a Minecraft server with RCON enabled.
