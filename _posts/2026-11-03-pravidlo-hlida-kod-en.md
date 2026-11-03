---
layout: post
lang: en
page_id: pravidlo-hlida-kod
title: "🛡️ A Rule Enforced by the Model Is a Wish"
description: "A rule enforced by the model is a wish. A rule enforced by code is a rule. Five places where trust leaks when you work with AI agents, and what to build there instead of another prompt."
date: 2026-11-03 12:00:00 +0100
categories: [Programming, AI]
tags: [ai, agents, claude, review]
image:
  path: /assets/img/posts/2026-11-03-pravidlo-hlida-kod-en.png
permalink: /posts/pravidlo-hlida-kod/
---

A rule enforced by the model is a wish. A rule enforced by code is a rule.

AI agents can handle a whole task these days. They read the brief, write the code and the tests, and fix whatever the review finds. So we trust them with more and more. And that makes every place where a check only looks like a check more expensive.

Here are five such places.

## 1. A ban that is only a request

You write in the agent's instructions: *"Never push without confirmation."* The agent obeys. Once, twice, ten times. Then one day it's rushing a fix and simply runs `git push`.

A prompt isn't a ban. It's a request. The model weighs it against everything else in its context, and sometimes the request loses. Does writing it in capitals help? Maybe a little. It still won't be a ban. Three exclamation marks? Go ahead, try.

What works is code. A hook or a permission rule that stops the agent before it writes anything out, pushes or touches production, or at least asks you first. Code doesn't have a bad day. It isn't in a hurry. You can't talk it round, though sometimes you can get around it. More on that at the end.

Keep the rules in the prompt. Just don't let them be the only thing standing between the agent and production.

**A prompt asks. Code enforces.**

## 2. The reviewer who only read

A second agent does the code review and approves a call to a method that doesn't exist. PHPStan would have caught it immediately. Except nobody ran it. The reviewer "checked the types by reading".

Or it notices that a test goes red on a newer PHP version and writes: *"That's an intentional safeguard for the future."* It doesn't look at the CI configuration. Pity. It would have seen that this future is already running in CI today.

The reviewer sees the diff. It has to look up the rest of the system on its own. And when nothing forces it to, it doesn't. So it should run the tools itself: static analysis, tests, every supported version.

The reviewer runs into something suspicious and waves it off: *"that's acceptable"*, *"that's intended"*, *"that won't happen in practice"*. Sometimes it's right. But this is how bugs slip through. Below is a case where the reviewer did exactly that three times. It may dismiss something like that only if it points to where it shows up: who calls the method, where its data comes from, what environment it runs in. If that can't be traced, it isn't "acceptable", it's an open question.

I took older bugs that review had let through and had them reviewed again with different briefs, three runs per brief:

- **A bug that only shows up on a newer PHP version.** Neither with the original brief, nor with *"be skeptical"*, nor with *"measure"* did the reviewer report it as a defect even once. Three times it noticed it and dismissed it with exactly that sentence about the safeguard. Only the brief *"measure and trace where it shows up"* caught it, and **in all three runs**: the reviewer opened the CI configuration and measured it.
- **An edge-case input that passes validation and then crashes the library.** Here, measuring was enough. With the brief *"measure"*, the reviewer reported it as a defect in two runs out of three. Without measuring, not once; at most it mentioned it as a hypothesis.

Three runs per variant isn't statistics, just a direction. But the direction is clear: "be skeptical" changes almost nothing.

**Skepticism without a tool is just stricter reading.**

## 3. A second opinion that isn't one

An agent finishes a change and reports: *"Only admins will see the Save button now."* A second agent is supposed to check. It opens the template. It sees the role condition, it sees the click handler. It agrees. And on the page the button works for nobody, because a script one line above crashes with an error. Nobody saw it, because nobody looked.

Two readings, one source. One verification with two signatures.

Verification has to come from somewhere other than the claim. What the user sees is proven by the rendered page: console, DOM, screenshot. How the code behaves is proven by a test or a measurement. And when a user reports something different from what the code says? The report wins, not the code. You can dispute it only once you've reproduced it yourself.

**Read twice is not verified twice.**

## 4. A "pass" that proves nothing

The agent reports: *"Tests pass, lint is clean."* They did pass. Three edits ago. Or it ran one tool and reported that the whole suite passed. Or the report lists what passed and stays silent about what never ran at all.

A status report looks like a fact, and nobody asks which run it came from. So it should always contain three things: which command the agent ran after the last edit, what that command printed on its last line, and what it didn't run at all. Even better in a fixed format, for example:

```text
type=report  task=3  commit=a1b2c3d
measured:  composer test → OK (120 tests, 342 assertions)
not run:   E2E tests – no database
```

`commit` says which state of the code was measured. If it differs from the latest commit, the report is stale. And a fixed format can be enforced by code: a report missing `measured` or `not run` gets sent back.

And when `not run` is missing or empty? That doesn't mean everything ran. It means something is being withheld.

**A report without output is just an impression.**

## 5. An assumption posing as a fact

The API returns `403` and the agent doesn't know why. So it guesses. In the summary and in a code comment it writes: *"The endpoint requires the admin role."* As a fact. Nobody said it, it isn't written anywhere. The next agent reads it in the comment, and from then on it's the truth as far as that agent is concerned.

An assumption written confidently looks like a finding. What the agent only infers belongs among the questions for a human, not in the text as a claim. Same with numbers: either measured, or marked as an estimate.

**Sounding sure is not the same as knowing.**

## One layer isn't enough

Looking back, I went through 18 pieces of work and counted the serious defects (bugs, security problems, gaps in tests) found in them: 150. **35 of them slipped through task review** and were caught only by a later layer, most often the final review of the whole branch. The remaining 115 were caught by task review. So it works, it just isn't enough on its own. The sample is small and I assigned some of the severities after the fact, so take the numbers as a rough guide.

## What this doesn't solve

No safeguard is free or complete: if it asks all the time, people learn to click through, and whatever can be done another way gets through. A sandbox helps here: the operating system then guards network and disk access, whatever tool the agent reaches for. But it too lets through anything you allow. And every extra check costs time and tokens.

## Checklist

- What the agent must not do on its own (writing out, pushing, production) is guarded by **a hook or a permission rule**, not a sentence in the prompt.
- The reviewer **runs the analysis and tests itself**. What it didn't run is a gap, not a "pass".
- The reviewer may dismiss a finding only if it **points to where it shows up**. Otherwise it's an open question.
- Verification comes **from somewhere other than the claim**: what the user sees, from the rendered page, not from the source.
- Status is reported as **the command after the last edit, its last line, and what didn't run**.
- An agent's assumption is **a question**, not a fact.
- **Write down** the bugs your checks let through. They make the best tests for your next reviewer brief.

**The prompt says what should happen. Whether it actually happened has to be verified by something else: code, a measurement, a second source, a human.**
