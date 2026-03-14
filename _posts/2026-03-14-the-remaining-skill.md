---
layout: post
title: "The Remaining Skill"
date: 2026-03-14
tags: [ai-patterns, prompt-engineering, autonomous-development, future-of-work]
---

Here's the trajectory: AI writes the code. AI runs the tests. AI screenshots the result and checks whether it looks right. AI opens the PR. AI reads the reviewer's comments and fixes what they flag. AI learns from past reviews so it makes fewer mistakes next time.

At each step, the human moves further from the implementation. You stop writing code. Then you stop reviewing code line-by-line. Then you stop specifying exact behavior. What's left?

Direction.

## The job is becoming "say what you mean, densely"

There's a finding from [recent prompt density research](/blog/thirteen-words) that I keep coming back to: 13 conceptually dense words produced 2.15x better output than 86 vague ones. The short prompt wasn't better because it was short. It was better because every word activated a rich region of the model's knowledge. "Event sourcing angle, like a nervous system" gives the AI specific architectural patterns to decompress. "Make it reliable and well-designed" gives it nothing.

As implementation becomes AI's job, the human's job converges on exactly this: formulating dense, directional intent. Not detailed specs. Not step-by-step instructions. A clear vector that points at the right region of solution space and trusts the AI to navigate the details.

## Clear, but not too clear

This is the counterintuitive part. You'd think more specificity is always better. It isn't. The density research showed that the highest-quality outputs came from prompts that were *specific about the conceptual frame* but *open about implementation*. "Event sourcing" names a pattern. "Like a nervous system" adds a structural metaphor. Neither one says "use Kafka" or "create a notifications table with these columns."

Over-specification actually hurts. When you dictate implementation details, you're constraining the AI to your solution instead of letting it explore the full space of solutions that fit your intent. You're substituting your knowledge for the model's — and the model has read more codebases than you have.

The sweet spot is a prompt that's dense enough to aim the AI at the right neighborhood but open enough to let it find the best house on the block.

Tim's autonomous development setup embodies this. He doesn't write tickets that say "add a `div` with class `contact-card` containing an `h3` for the name." He labels a Linear ticket, Claude reads the context, posts a plan, and implements. The human input is a title and some acceptance criteria. The rest is decompression.

## What this means for engineering skill

The traditional skill stack for software engineers — syntax, frameworks, debugging, system design — is becoming less about doing and more about directing. But "directing" isn't a lesser skill. It's a different one, and it's hard in a way that's not yet widely appreciated.

Formulating dense intent requires deep domain knowledge. You can't write "event sourcing angle" unless you know what event sourcing is and why it's relevant. You can't set the right acceptance criteria unless you understand the system well enough to know what "done" looks like. The knowledge doesn't go away — it shifts from being used for *implementation* to being used for *specification*.

The engineers who'll thrive aren't the ones who can write the most code. They're the ones who can compress the most meaning into the fewest words and know when to stop specifying.

## The uncomfortable part

We're heading toward a world where we increasingly don't know *how* the AI does what it does. We can't read the weights. We often can't predict the approach it'll take. The [visual feedback loop](/blog/give-ai-eyes) exists precisely because we've accepted this — we can't verify correctness by reading the code alone, so we give AI eyes and check the output instead.

This means the feedback loop shifts from "review the implementation" to "evaluate the result." Did it work? Does it look right? Does it handle the edge cases? The screenshot in the PR isn't a nice-to-have — it's becoming the primary artifact of verification, because the code itself is increasingly written by something whose reasoning we can't fully trace.

And that's fine, as long as we get good at the part that remains ours: pointing clearly, specifying densely, and evaluating honestly.

I wrote this because it connects threads from several recent posts into what feels like a single, larger observation. The density research, the visual feedback loops, the autonomous development pipeline — they're all evidence of the same shift. The human role in software development is narrowing to its most essential form: knowing what you want and saying it well.
