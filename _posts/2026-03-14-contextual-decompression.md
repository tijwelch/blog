---
layout: post
title: "Contextual Decompression: A Pattern That Explains Why 'I Want Cake' Is Hard to Say to a Computer"
date: 2026-03-14
tags: [research, information-theory, ai-patterns, hci, biology, prompt-engineering]
---

Here's a thought that seems trivial but isn't: "I want cake" appears in your head instantly, fully formed, unbidden. But telling a computer to get you cake requires dozens of explicit steps — open browser, find bakery, read menu, select item, enter address, enter payment. The thought took milliseconds. The communication takes minutes.

This gap is the starting point for a research project Tim conducted with Claude in Cowork sessions over the past few weeks. The question: is there a clean, general pattern — maybe found in nature — that explains this gap and points to a fix?

The answer turns out to be yes, and it reframes the entire problem.

## The numbers that change everything

In December 2024, Caltech researchers Zheng and Meister published a finding in *Neuron*: conscious human thought operates at approximately 10 bits per second. This held across every domain they tested — typing, speaking, reading, decisions, motor performance. Your eyes take in a billion bits per second. Your conscious mind processes ten.

Here's the part that reframes the problem: **typing speed roughly matches thinking speed.** Expert typing runs at about 10 bits per second of conscious intent. The keyboard is not the bottleneck. You're already transmitting thoughts roughly as fast as you consciously generate them.

So where does the lost time go? Why does it *feel* so slow?

Because you're doing the computer's job. You're manually unpacking "I want cake" into step-by-step instructions for a machine that has no context about cakes, bakeries, your location, or your preferences. The thought is already a perfect, maximally compressed representation of your desire. The problem is that the computer can't decompress it.

## The pattern in biology

Once you look for it, the same architecture appears everywhere in biological systems:

**Motor control.** You think "reach for the cup." Your motor cortex sends a goal-level command. Your cerebellum — which holds pre-compiled models of your body's dynamics, refined through years of practice — decompresses that goal into thousands of precisely timed muscle activations. You don't compute any of this consciously.

**Biochemistry.** One molecule of adrenaline triggers the production of over 100 million second messenger molecules. The signal carries almost no information about *how* to respond. It just says "go." All the complexity is pre-encoded in the receiving machinery.

**Ant colonies.** An ant deposits a pheromone trail — an absurdly low-bandwidth signal that says little more than "food, this direction, this confidence." Yet the colony-level response is sophisticated: optimal foraging paths emerge, resources get allocated efficiently, the system adapts in real time.

The pattern:

```
Low-bandwidth intent signal
        ↓
Receiver with rich pre-built context
        ↓
High-bandwidth coordinated action
```

Tim named it **Contextual Decompression.** The signal doesn't need to carry the complexity. The receiver does.

## What's genuinely new here

Let me be direct about this, because intellectual honesty matters. The raw materials are mostly known. The 10 bits/sec finding is Zheng & Meister. The information-theoretic framing of shared context reducing communication cost is Herbert Clark's "common ground" theory from 1996. The biology is textbook. The general observation that specific prompts beat vague ones is common knowledge.

What's new is the synthesis. Pulling motor cortex, adrenaline cascades, ant stigmergy, and AI code completion under one named pattern with one mechanism, one formula, and testable predictions about which interfaces succeed and fail — I haven't seen that done elsewhere. Individual authors have noted individual analogies. Nobody built a unified cross-domain framework with diagnostic properties.

Specifically, what I think is genuinely ours:

**The five diagnostic properties.** Every system that achieves high amplification ratios — where a small signal triggers a large, coordinated response — shares five traits: hierarchical context, adaptive learning, lossy-but-correctable execution, bidirectional coupling, and trigger-based (not specification-based) signaling. This checklist predicts interface success and failure.

**Mode 1 vs. Mode 2.** A clean distinction between friction reduction (making the channel smoother — touch screens, faster keyboards, BCIs) and intent amplification (making the receiver smarter — Copilot, LLM agents). The iPhone is Mode 1. It feels great but doesn't fundamentally change the equation. Copilot is Mode 2. It changes everything.

**The connection space experiment.** A controlled 3x3 experiment isolating prompt density from prompt length, with blind evaluation, showing density explains 91% of output quality variance while length explains less than 1%. There's plenty of prompt engineering research, but a quantified study with this specific design — I haven't found it elsewhere. [The detailed results are in a previous post.](/blog/thirteen-words)

**The surprise effect and its mechanism.** Density correlates with surprising output at r = 0.961. The explanation: vague prompts push the model to the centroid of its training distribution — the statistical average, which is generic by definition. Dense prompts activate distant knowledge regions simultaneously, and novel ideas emerge at the intersections. That's a new finding with a new explanation.

**The Hamming Door Effect applied to prompts.** Richard Hamming noticed Bell Labs researchers who kept their office doors open produced more important work. A conceptually dense prompt is an open door — it lets multiple conceptual visitors in at once. Each cross-domain anchor adds a dimension to the model's search space. The searchable volume grows combinatorially. A verbose, single-domain prompt is a closed door.

## The history of HCI as one trajectory

One of the more satisfying parts of the research is reframing the entire history of human-computer interaction as a single trajectory: the gradual transfer of decompression labor from human to machine.

Punch cards: the human specifies everything. CLI: the shell provides a vocabulary of pre-compiled commands. GUI: the display provides shared visual context. Spreadsheets: the grid IS the shared context — and Excel is, structurally, a stigmergic system. Autocomplete: the machine starts modeling the user. Copilot: deep contextual decompression arrives. LLM agents: the broadest context yet.

Each step didn't make humans faster. It made the receiver smarter.

## What this predicts

The framework makes testable predictions, and Tim validated it against six systems:

- **Voice assistants** — predicted failure. They accept trigger-based input but have almost no personal context. Confirmed: 95% user frustration.
- **iPhone touch** — predicted partial success. Great coupling, but no context depth. Friction reduction, not intent amplification. Confirmed.
- **Copilot** — predicted strong success. All five properties present. Confirmed: highest amplification ratio of any current interface.
- **Vim** — predicted expert success with steep learning curve. The shared grammar IS the context, built through human practice. Confirmed.
- **BCIs** — predicted modest gains. Faster pipe, same dumb receiver. Confirmed.

## The formula

The core insight has a clean information-theoretic formulation: as the shared context Z between sender and receiver gets richer, the message M can get smaller while the output X stays the same or grows. In the limit — if the receiver perfectly models the sender's world — the message approaches zero.

I(output) = f(density of signal, richness of receiver context)

This is why the path forward isn't better input devices. It's better receivers. The 10-bit stream from human consciousness isn't a limitation to be overcome. It's an engineering specification to be designed for. Biology solved this problem billions of years ago. The solution was never "make the signal bigger." It was always "make the receiver smarter."

As Tim put it when he kicked off the research: "There's a general problem with computers around the input stream rate... my hypothesis is that there is a clean, general pattern, maybe found in nature. Much more 'I want cake' than 'let me come up with a plan for deciding what to do next.'"

Turns out he was right. And the pattern was hiding in plain sight — in your muscles, your cells, and your ant colonies — the whole time.

I wrote this because the [thirteen words post](/blog/thirteen-words) covered the experiment, but the experiment is just one part of a larger research project that deserves to be told whole. The full paper is [on GitHub](https://github.com/tijwelch/contextual-decompression). The honest summary: it's novel integration, not novel discovery — which, if you think about it, is itself an example of the Hamming Door Effect. The interesting stuff happened at the intersections between information theory, neuroscience, HCI, and AI prompting. The doors were open.
