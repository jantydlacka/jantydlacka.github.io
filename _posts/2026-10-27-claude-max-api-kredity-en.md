---
layout: post
lang: en
page_id: claude-max-api-kredity
title: "💵 Got Claude Max? There's $100 a Month of API Credit Waiting for You"
description: "Claude Max 5x and 20x include $100 or $200 a month for the Claude API. You just have to claim it. No card needed, and it doesn't cover Claude Code."
date: 2026-10-27 22:00:00 +0100
categories: [Programming, AI]
tags: [claude, ai, api]
image:
  path: assets/img/posts/2026-10-27-claude-max-api-kredity.png
permalink: /posts/claude-max-api-kredity/
---

You may be leaving **$100 a month in API credit** on the table. Or $200, depending on your plan. All it takes is paying for Claude Max and never claiming it.

## How much, and for whom

| Plan | Monthly credit |
|---|---|
| Max 5x | $100 |
| Max 20x | $200 |
| Team | $20 per Standard seat, $100 per Premium seat, pooled and capped at $500 |
| Free, Pro, Enterprise | none |

## It won't show up on its own

On claude.ai, open **Settings → Billing** (on Team: **Organization settings → Billing**), pick an organization in the [Claude Console](https://platform.claude.com/){:target="_blank" rel="noopener"} under *API credits* (or create one), accept the terms, done. The credit shows up in the Console as *Promotional credits*. You're eligible after seven days on the plan.

One thing to watch: **whichever organization you pick is where the credit stays.** A subscription links to just one, and only support can move it. Got a company organization in the Console too? Careful you don't gift the credit to your coworkers 😉

## Keep your card in your wallet

After you claim the credit, the Console may offer to add a payment card. You don't have to. Without one, the API simply stops when the credit runs out and waits for the next billing cycle. The docs put it bluntly: *"Usage is never charged to your Claude plan."* You only start paying if you buy credits yourself or turn on auto-reload.

![Max 5x subscription with a $100 monthly API credit: link once, no card, the credit does not cover Claude Code](assets/img/posts/2026-10-27-claude-max-api-kredity-detail.png)

## The credit is for the API only

You can spend the credit on the Claude API, Managed Agents, the Agent SDK and the Playground. Not on Claude Code: interactive work in the terminal, IDE, desktop or web keeps running on your subscription, with the same limits. What matters is how you start Claude, not what it's built on. `claude -p` with an API key from your linked organization does draw on the credit (it's billed as Agent SDK usage). The Claude Code GitHub Action or an IDE extension doesn't, not even with `-p`.

So be careful where you store your API key. If you put it in the `ANTHROPIC_API_KEY` environment variable (in `~/.bashrc`, `~/.zshrc` or your Windows settings), Claude Code will find it. In interactive mode it asks once, and if you approve, it [starts using the key instead of your subscription](https://code.claude.com/docs/en/iam#authentication-precedence){:target="_blank" rel="noopener"}. But the Max credit doesn't cover interactive Claude Code, so you end up with *Credit balance too low*. And `claude -p` doesn't even ask: it takes the key right away and starts eating into your API credit instead of your subscription limits. Keep the key in the config of the app that needs it, not in your environment variables.

## So what's it good for?

For everyday work with Claude, the subscription is all you need. The API is for when Claude should do the work without you: a script in cron, an AI feature in your own app, an agent built on the Agent SDK. For experiments and smaller projects, a hundred dollars a month is plenty, in my opinion.

So go claim it. Whatever you don't use expires at the end of the cycle.

Full terms: [API credits for Max and Team plans](https://platform.claude.com/docs/en/about-claude/api-credits-for-subscribers){:target="_blank" rel="noopener"} and the [Help Center FAQ](https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans){:target="_blank" rel="noopener"}.
