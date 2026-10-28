---
layout: post
lang: cs
page_id: claude-max-api-kredity
title: "💵 Máte Claude Max? Pak na vás čeká 100 $ měsíčně na API"
description: "Předplatné Claude Max 5x a 20x obsahuje měsíční kredit 100 nebo 200 USD na Claude API. Jen si ho musíte vyzvednout. Kartu nepotřebujete, na Claude Code neplatí."
date: 2026-10-27 22:00:00 +0100
categories: [Programming, AI]
tags: [claude, ai, api]
image:
  path: /assets/img/posts/2026-10-27-claude-max-api-kredity.png
---

Možná každý měsíc propásnete **100 dolarů na API**. Nebo dvě stě, podle plánu. Stačí platit Claude Max a nikdy si pro ně nedojít.

## Kolik a komu

| Plán | Kredit měsíčně |
|---|---|
| Max 5x | 100 USD |
| Max 20x | 200 USD |
| Team | 20 USD za místo Standard, 100 USD za místo Premium, dohromady nejvýš 500 USD |
| Free, Pro, Enterprise | nic |

## Kredit si musíte vyzvednout

Na claude.ai otevřete **Settings → Billing** (u Team: **Organization settings → Billing**), v sekci *API credits* vyberete organizaci v [Claude Console](https://platform.claude.com/){:target="_blank" rel="noopener"} (nebo založíte novou), odkliknete podmínky a je to. Kredit se objeví v Console jako *Promotional credits*. Kdo si Max předplatil teprve nedávno, dostane nárok sedm dní po začátku předplatného.

Pozor na jednu věc: **kterou organizaci vyberete, u té kredit zůstane.** Předplatné jde napojit jen na jednu a přehodit ho jinam umí jen podpora. Máte v Console i firemní organizaci? Tak ať ten kredit nedarujete kolegům 😉

## Kartu nechte v peněžence

Console vám po vyzvednutí může nabídnout přidání platební karty. Nemusíte. Bez ní se API po vyčerpání kreditu prostě zastaví a počká na další fakturační období. Dokumentace to říká bez okolků: *„Usage is never charged to your Claude plan.“* Platit začnete, jen pokud si sami dokoupíte kredit nebo zapnete auto-reload.

![Předplatné Max 5x s kreditem 100 USD měsíčně na API: propojit jednou, bez karty, kredit nepokrývá Claude Code](assets/img/posts/2026-10-27-claude-max-api-kredity-detail.png)

## Kredit je jen na API

Kredit můžete utratit za Claude API, Managed Agents, Agent SDK a Playground. Na Claude Code ne: interaktivní práce v terminálu, IDE, desktopu nebo na webu dál jede z předplatného a jeho limity zůstávají stejné. Rozhoduje přitom, jak Claude spustíte, ne na čem je postavený. `claude -p` s API klíčem z propojené organizace kredit čerpá (počítá se jako Agent SDK). GitHub Action s Claude Code nebo rozšíření v IDE ne, a to ani s `-p`.

Pozor proto, kam uložíte API klíč. Když ho dáte do proměnné prostředí `ANTHROPIC_API_KEY` (v `~/.bashrc`, `~/.zshrc` nebo v nastavení Windows), Claude Code si ho najde. V interaktivním režimu se jednou zeptá, a když to odkliknete, [začne ho používat místo předplatného](https://code.claude.com/docs/en/iam#authentication-precedence){:target="_blank" rel="noopener"}. Jenže interaktivní Claude Code kredit z Max nekryje, takže skončíte na hlášce *Credit balance too low*. A `claude -p` se ani neptá: vezme klíč rovnou a místo limitů předplatného začne ujídat váš API kredit. Klíč proto patří do konfigurace aplikace, která ho potřebuje, ne do proměnných prostředí.

## A k čemu mi to je?

Na běžnou práci s Claudem vám předplatné stačí. API se hodí tam, kde má Claude makat bez vás: skript v cronu, AI funkce ve vlastní aplikaci, agent na Agent SDK. Na hraní a menší projekty je podle mě sto dolarů měsíčně až dost.

Tak si pro ně dojděte. Co nevyčerpáte, na konci období propadne.

Všechny podmínky: [API credits for Max and Team plans](https://platform.claude.com/docs/en/about-claude/api-credits-for-subscribers){:target="_blank" rel="noopener"} a [FAQ v Help Center](https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans){:target="_blank" rel="noopener"}.
