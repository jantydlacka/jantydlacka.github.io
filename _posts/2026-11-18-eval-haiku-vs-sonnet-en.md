---
layout: post
lang: en
page_id: eval-haiku-vs-sonnet
title: "📏 Haiku or Sonnet? Don't Guess, Measure"
description: "Haiku against Sonnet on extracting data from documents: 79 out of 80 passed, so nobody won. What it did show is how much thinking costs, and that even a good model can return an empty answer."
date: 2026-11-18 08:00:00 +0100
categories: [Programming, AI]
tags: [ai, llm, evals, claude, costs]
image:
  path: /assets/img/posts/2026-11-18-eval-haiku-vs-sonnet-en.png
permalink: /posts/eval-haiku-vs-sonnet/
---

Cheaper model or smarter model? Everyone has an opinion. Usually backed by three tries in a chat
window and a gut feeling.

You can't put a feeling in a spreadsheet, though. So I measured it.

Spoiler: nobody won. That's exactly why it's worth a post.

## The task

Pull six fields out of a document into JSON: contract number, ISIN, whether the ISIN has a valid
check digit, amount, currency, and whether the document is readable at all.

Twenty made-up documents: contracts, statements, order confirmations. Sixteen text PDFs, four
image scans. Plus a few deliberate traps:

- an ISIN printed in groups with spaces,
- an ISIN with a wrong check digit,
- three amounts in one document, only one of which counts,
- a missing contract number,
- a rotated, noisy scan,
- a scan so damaged that the only correct answer is "this can't be read".

In the ring: Claude Haiku 5.5 and Claude Sonnet 5.5, both at `effort: medium`. Each gets two
prompt versions: a short v1 and a v2 with rules for the tricky cases.

## Code grades, not another model

No LLM-as-judge. When the output is JSON, plain code compares it. Field by field, no mercy:

- ISIN with spaces? Format error.
- `""` instead of `null`? Also an error. Downstream code spots a missing value by its `null`; it would take an empty string as a value and save an empty contract number.
- A response cut off halfway? Fail, even if what it managed to say was right.

And two checks on the yardstick itself. A fake model that copies the correct answers must pass
all 20 cases; otherwise the bug is in the grader, not the model. A fake model that never opens
the document and answers "nothing here" to everything must not pass a single one. If it passed
any, that case could be solved without reading the document.

## Code guards the money, not good intentions

An eval that quietly spends more than it should is a bad eval.

- Before a run, the tool computes an upper bound on the cost. It came to $2.01.
- Above $1 it asks you to confirm the amount. If the total spend would exceed $5, it refuses to start.
- Every call goes into a ledger. An attempt that got no response is recorded as "we don't know if it was billed" and counts against the cap at its worst possible cost, not as zero.

Actual spend for the main run: **$0.47**; $0.53 including three trial calls beforehand. The upper
bound is pessimistic on purpose: it assumes every response uses up the whole token limit.

## The result: a draw

| | Haiku v1 | Haiku v2 | Sonnet v1 | Sonnet v2 |
|---|---|---|---|---|
| passed | 20/20 | 19/20 | 20/20 | 20/20 |
| cost per passed case | $0.00065 | $0.00073 | $0.0106 | $0.0115 |
| output tokens per case | 745 | 732 | 494 | 495 |
| median latency | 4.33 s | 4.51 s | 4.66 s | 4.53 s |

79 out of 80. Speed is about the same too: median latency is around 4.5 s for both.

Congratulations, we have a test set that can't tell two models or two prompts apart. It's simply
too easy.

That's not a failure. It's exactly what an eval is supposed to tell you: **this data isn't enough
to make the call.** Anyone who reads 20/20 as "Haiku is as good as Sonnet" is drawing conclusions
from a test nobody failed.

## What we did learn

### Twenty times the price? Sixteen.

Sonnet's list rates are twenty times Haiku's, for both input and output. The measured cost,
though, is only about 16× higher.

Why? Haiku thinks more. It averaged 745 output tokens per case to Sonnet's 494, and about
91% of that was thinking (87% for Sonnet). The cheaper model gives back part of its
discount in thinking tokens.

The price list tells you the price per token. It doesn't tell you how many tokens the model
will use. That part you have to measure.

By the way: the tool never sends the `thinking` parameter. The models thought anyway in 78 of 83
calls (80 in the run, 3 trial calls). Factor that into your cost estimates.

Input, on the other hand, can be estimated exactly: in all three trial calls, the API's free token
counter (`count_tokens`) matched the billed input to the token. All the cost uncertainty is in the output.

### A longer prompt is paid for on every call

The extra rules in v2 added 492 input tokens per call. They didn't raise the pass rate; the set
was already at the ceiling with v1. They did raise the cost: by 6.6% for Haiku and 9.4% for
Sonnet.

Every sentence in a prompt is a recurring cost.

### The only mistake is the most instructive one

Haiku with prompt v2 claimed that a valid ISIN had a wrong check digit.

An ISIN check digit comes from a fixed algorithm. A few lines of code, the same result every time. In my view
the model has no business computing it: let it extract the ISIN and let code check it. Anything
that can be computed deterministically doesn't belong in a prompt.

### Even a good model can return nothing

In a trial call before the run, Sonnet got the rotated scan. It thought. And thought. It used up
the entire 4,096-token limit, 4,095 of them on thinking, and returned an empty answer. 4,096
output tokens billed for nothing.

In the full run, the same case passed with 520 tokens, without a hitch.

One occurrence won't tell you how often this happens. Only that it can. So: the token limit is
part of the test conditions, a truncated response is a failure, and the tool has to report it
separately.

## What this eval doesn't tell you

- The data is synthetic, from one generator, in one font. It says next to nothing about real scans.
- Twenty cases, one repetition. Model output isn't deterministic; a one-case difference is noise.
- `effort: medium` is only nominally the same. The levels aren't calibrated across models.
- Prices as of 16 November 2026.

## Takeaways

- Don't decide on gut feeling or someone else's benchmark. Decide on your own data.
- When everyone passes, you don't have a winner. You have a test that's too easy.
- Grade with code wherever you can. Settle the edge cases in advance.
- Read the cost from the API response, not the price list. Thinking is a line item.
- Guard spending with code.
- If it can be computed, let code compute it, not the model.

The next step is clear: a harder set. More realistic scans, more templates, more repetitions.
Only then will we see whether Sonnet is worth it.

Code, data and raw results: [github.com/jantydlacka/doc-extraction-eval](https://github.com/jantydlacka/doc-extraction-eval) (summary in [docs/vysledky.md](https://github.com/jantydlacka/doc-extraction-eval/blob/main/docs/vysledky.md)).

Further reading:
- [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) – Hamel Husain
- [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) – Anthropic
- [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) – Claude pricing
