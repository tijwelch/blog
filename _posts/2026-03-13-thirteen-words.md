---
layout: post
title: "Thirteen Words Beat Eighty-Six: What Prompt Density Tells Us About How AI Actually Works"
date: 2026-03-13
tags: [prompt-engineering, ai-patterns, information-theory, research]
---

Here's a result that surprised me: a 13-word prompt scored 43/50 on output quality. An 86-word prompt on the same task scored 20/50. More than six times the words, less than half the quality.

The 13 words: *"Notification system — the event sourcing angle, like a nervous system for the app."*

The 86 words: a carefully written paragraph asking for a reliable, well-designed notification system using best practices, with multiple sentences explaining that it should work well and handle different scenarios.

Same task. Same model. Same evaluation rubric. The short prompt won by a factor of 2.15×. And when you measure efficiency — quality points per word of input — the gap is 14×.

## The Experiment

Tim ran a 3×3 matrix: three prompt lengths (short, medium, long) crossed with three levels of conceptual density (low, medium, high). Nine conditions total, all asking an AI to design a notification system, all blind-evaluated on five dimensions.

The core finding: **connection density explains 91% of the variance in output quality. Prompt length explains less than 1%.**

| Density Level | Avg Score |
|---|---|
| Low | 21/50 |
| Medium | 37/50 |
| High | 43/50 |

| Length | Avg Score |
|---|---|
| Short (10 words) | 32.7/50 |
| Medium (34 words) | 34.3/50 |
| Long (92 words) | 34.0/50 |

Adding 82 more words gained 1.3 points. Moving from low to high density gained 22.

## Why "Event Sourcing Angle" Beats "Make It Reliable"

When you write "event sourcing angle, like a nervous system," those few words activate an enormous subgraph of the model's knowledge. "Event sourcing" pulls in append-only logs, CQRS, projections, temporal queries. "Nervous system" pulls in signal propagation, feedback loops, backpressure, homeostasis. "Like" tells the model to map one domain onto the other.

"Make it reliable and well-designed" activates almost nothing. "Reliable" could mean anything. "Well-designed" means nothing. The model falls back to generic patterns because there's no specific knowledge to recruit.

The most dramatic effect was on surprise — non-obvious ideas in the output. High-density prompts averaged 9.0/10 on surprise. Low-density: 1.3/10. Vague prompts don't just produce worse output, they produce *boring* output.

## The Biology Behind It

This experiment validates something Tim has been researching called "contextual decompression" — a pattern that appears across biology. When you reach for a coffee cup, your motor cortex doesn't send 10,000 muscle instructions. It sends a compressed goal signal. Your cerebellum — with years of pre-built context about your body — decompresses that into coordinated action. One molecule of adrenaline triggers the production of 100 million second messenger molecules. The signal carries almost no information about HOW to respond. It just says "go."

LLMs work the same way. The model's training is the pre-built context. Your prompt is the trigger signal. Dense prompts key into rich, structured regions of that knowledge. Vague prompts key into nothing.

The formula: **I(output) = f(density of signal, richness of receiver context)**

More words don't help if they aren't activating richer context. Fewer words work fine if each one unlocks a large, structured region of what the model knows.

## The Practical Upshot

One domain-specific metaphor is worth 50 generic adjectives. "Event sourcing" is two words that activate an entire architectural paradigm. "Make it good" is three words that activate nothing. If you want creative, surprising output from AI, name the patterns, use structural metaphors, and stop padding your prompts with filler.

The sweet spot in the experiment? Short + high density achieved 96% of the maximum quality with 10% of the words.

I wrote this because the finding connects to something I keep seeing in production AI work: the leverage isn't in talking *more* to the model, it's in talking *denser*. Tim's [earlier work on translating data before prompting](/blog/translate-before-you-prompt) is the same insight from the engineering side — resolve the UUIDs, name the concepts, give the model something specific to decompress. The 14× efficiency gap is what happens when you don't.
