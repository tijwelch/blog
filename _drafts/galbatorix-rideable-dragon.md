---
layout: post
title: "An 11-Year-Old and a CLI Built a Rideable Three-Headed Dragon in Vanilla Minecraft"
date: 2026-08-30
tags: [minecraft, claude-code, mineclawd, agents, datapacks]
---

An 11-year-old sat down at Tim's computer, opened a terminal running Claude Code, and pasted in a prompt he'd written himself. He'd never used a command line before. He'd never written code. The prompt was a fever dream: a three-headed cosmic dragon overlord named Galbatorix, made of crying obsidian and nebula patterns, with galaxy-swirl eyes, crystalline horns, four arms, a tail that splits into glowing tendrils — rideable, flyable, with speed boost and attacks. The kind of thing a kid comes up with when nobody's told him what's supposed to be hard.

Over the next two hours, he and Claude Code built the whole thing. In vanilla Minecraft. No mods.

Here's what actually happened, technically.

## The Setup

Mineclawd is a CLAUDE.md file and a Python RCON client under 200 lines. That's the entire product. The CLAUDE.md teaches Claude Minecraft's coordinate system, block palette, fill command limits, and building patterns. The RCON client opens a socket to the Minecraft server and sends commands. Claude reads the instructions, writes commands, executes them via RCON, and checks the results. There's no plugin, no API wrapper, no framework. It's a prompt and a socket.

## The Static Build

The kid's first prompt produced a 400-line Python script that built Galbatorix as a statue, block by block, in real time while he watched in Minecraft. Three heads made of crying obsidian with galaxy-swirl eyes of sea lanterns. Wings spanning 120 blocks. A full cosmic arena around it — orbiting planets, a black hole with an accretion disk, lightning arcs, cosmic breath beams firing from all three mouths. The camera orbited the finished build automatically.

Then he said what any kid would say: "Can I ride it?"

## Four Architecture Failures

What followed was a genuine engineering exploration. Claude tried four fundamentally different approaches before finding one that worked:

**Attempt 1: Wither core with display entities and chase AI.** A wither boss as the rideable base, surrounded by `block_display` entities forming the dragon's body. Failed because of a subtle bug: `execute unless entity ... run return 0` was halting the entire tick function even when entities existed. The `return 0` executed in a context where it killed the whole function chain.

**Attempt 2: Marker core with iron golem hitbox and horse mount.** Switched to an iron golem (neutral mob, survives peaceful mode — hostile mobs like slimes get despawned). But the golem's physical collision box trapped the horse underneath it. The kid said "I went that it was trapped under the whole rest of the mob."

**Attempt 3: Simplified horse with display entities.** Stripped everything down. But horses with `NoAI:1b` ignore player input entirely. Without it, they wander randomly. Running `data merge {NoAI:0b}` every tick to toggle it reset the horse's movement state. The kid: "now it cant move like its being crushed under a ton of weight." Then the horse levitated from follow_owner teleporting toward a creative-mode flying player, and died.

**Attempt 4: No mount entity.** This was the breakthrough. The kid said "do you think it is just too complex?" — and Claude scrapped the entire mount-entity approach. New architecture: the player *is* the dragon.

Every one of these failures happened live, with an 11-year-old watching blocks appear and disappear and debugging by describing what he saw on screen. "It's not moving." "I think the horse is trapped." "It is levitating in the air." He had no idea he was navigating entity AI systems, collision mechanics, and the fundamental constraint that Minecraft can't teleport entities with passengers. He just described what was wrong.

## The Architecture That Worked

The final system is a Minecraft datapack — a set of `.mcfunction` files that run server-side.

**~65 `block_display` entities** form the dragon's body. These are entities that render a block at a position with arbitrary scale, rotation, and translation. No collision, no AI, no hitbox. Three heads (one snarling, one calm, one mid-roar), wings with bone structure and translucent amethyst membrane, four arms with claws, legs, a segmented tail with glowing tendril tips, galaxy swirl on its chest, cosmic energy veins, crystalline horns. Each one is a single `summon` command with a transformation matrix.

**A tick function** runs 20 times per second. When the player has the `galba.flying` tag, every display entity teleports to the player's position:

```mcfunction
execute if entity @p[tag=galba.flying] as @e[tag=galba.part] at @p run tp @s ~ ~ ~ ~ ~
```

**Flight is look-direction-based.** Every tick, the player is teleported forward in their facing direction:

```mcfunction
execute as @a[tag=galba.flying,scores={galba.boost=0}] at @s
  if block ^ ^ ^2 air if block ^ ^1 ^2 air
  if block ^ ^ ^3 air if block ^ ^1 ^3 air
  run tp @s ^ ^ ^3
```

That `^ ^ ^3` is the key. Caret notation in Minecraft means "relative to where I'm looking." Look up, you climb. Look down, you dive. Look forward, you cruise at 3 blocks per tick — 60 blocks per second. No elytra, no creative flight, no mods. Just a teleport running 20 times per second.

**Collision detection** was added after the kid flew through the ground and died. ("I died in the underworld.") The `if block ^ ^ ^N air` checks at feet and head level prevent the teleport if terrain is ahead. Simple, but it works.

**Speed boost via item detection.** The kid wanted to go "faster than a happy ghast." Claude tried sneak-based speed tiers first — didn't work because the constant teleporting overrides the sneak state. Switched to using a renamed warped_fungus_on_a_stick as a "Cosmic Surge" item. Right-clicking it increments a scoreboard objective:

```mcfunction
scoreboard objectives add galba.boost minecraft.used:minecraft.warped_fungus_on_a_stick
```

When `galba.boost` is ≥1, flight speed jumps to 10 blocks per tick (200 blocks/second) with sonic boom particles and cosmic trails streaming off the wings.

**Triple fireball attacks.** A carrot_on_a_stick triggers the same way. Three `small_fireball` entities spawn offset to match the three heads — center, left, right — with dragon breath, soul fire, and flame particles:

```mcfunction
execute at @s anchored eyes run summon small_fireball ^ ^ ^3
execute at @s anchored eyes run summon small_fireball ^-2 ^1 ^3
execute at @s anchored eyes run summon small_fireball ^2 ^1 ^3
```

**Pet behavior when dismounted.** The dragon drifts toward the player at 0.2 blocks/tick, teleports to catch up if you get more than 30 blocks away:

```mcfunction
execute unless entity @p[tag=galba.flying] as @e[tag=galba.part,limit=1] at @s
  if entity @p[distance=5..] facing entity @p feet
  run tp @e[tag=galba.part] ^ ^ ^0.2
```

## What the Conversation Looked Like

The kid's messages read like this:

- "I want it to fly super fast, like faster than a happy ghast"
- "I died in the underworld. Can you make it to where he does cant go through blocks"
- "Can you also make it follow me while I am dismounted so it doesnt go away"
- "Do I look like I'm riding on a 3 headed dragon when people look up at me"

Each request took Claude about a minute to implement, test, and deploy to the running server. He never looked at the code. He never asked what a datapack was, what a scoreboard objective was, what a tick function was. He described what he wanted, described what went wrong, and iterated until it worked.

When things broke — and they broke constantly (horses trapped under mobs, entities levitating and dying, the player falling through the earth, display entities that refused to render, potion effects appearing from nowhere) — he just said what he saw. "There is some potion effect on my character, I'm seeing those ring things. Turn that off." Claude ran `effect clear @a`.

## What's Interesting Here

The datapack the kid ended up with is a real piece of engineering. 65 synchronized display entities. Look-direction flight via tick-rate teleportation. Item-based input detection using scoreboard objectives. Collision detection. State management (mounted/dismounted) with tag toggling. Pet AI with distance-based teleportation. Three-head attack offset calculation using anchor-relative coordinates.

None of that was visible to the person building it. The kid had an idea — the most maximalist, kitchen-sink, action-figure-back-of-the-box idea possible — and shipped it in an afternoon, in vanilla Minecraft, without writing or reading a single line of code.

His words after the first session: "A LOT of people would use this."

Maybe. What stuck with me is that the whole experience felt less like programming and more like directing. Not "write me a function." More "I died in the underworld — fix it." The gap between having an idea and seeing it exist in a 3D world was a conversation. That's what Mineclawd is.
