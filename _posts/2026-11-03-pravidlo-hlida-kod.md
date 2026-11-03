---
layout: post
lang: cs
page_id: pravidlo-hlida-kod
title: "🛡️ Pravidlo, které hlídá model, je přání"
description: "Pravidlo, které hlídá model, je přání. Pravidlo, které hlídá kód, je pravidlo. Pět míst, kudy při práci s AI agenty uniká důvěra, a co tam postavit místo dalšího promptu."
date: 2026-11-03 12:00:00 +0100
categories: [Programming, AI]
tags: [ai, agents, claude, review]
image:
  path: /assets/img/posts/2026-11-03-pravidlo-hlida-kod.png
---

Pravidlo, které hlídá model, je přání. Pravidlo, které hlídá kód, je pravidlo.

AI agenti dnes zvládnou celý task. Přečtou zadání, napíšou kód i testy a opraví, co najde recenze. Proto jim svěřujeme čím dál víc. A tím dráž stojí každé místo, kde kontrola jen vypadá jako kontrola.

Tady je pět takových míst.

## 1. Zákaz, který je jen prosbou

Napíšete agentovi do instrukcí: *„Nikdy nepushuj bez potvrzení.“* Agent poslechne. Jednou, podruhé, podesáté. A pak jednou spěchá s opravou a `git push` prostě pustí.

Prompt totiž není zákaz. Je to prosba. Model ji váží proti všemu ostatnímu, co má v kontextu, a občas prosba prohraje. Pomůže, když ji napíšete velkými písmeny? Možná trochu. Zákaz z ní nebude. Tři vykřičníky? Zkuste to.

Co funguje, je kód. Hook nebo pravidlo oprávnění, které agenta před zápisem ven, pushem nebo sahnutím na produkci zastaví, nebo se vás aspoň zeptá. Kód nemá špatný den. Nespěchá. Přemluvit ho nejde, obejít někdy ano – o tom na konci.

Pravidla v promptu klidně nechte. Jen ať už nejsou jediné, co stojí mezi agentem a produkcí.

**Prompt prosí. Kód hlídá.**

## 2. Recenzent, který jen četl

Druhý agent dělá code review a schválí volání metody, která neexistuje. PHPStan by to chytil hned. Jenže ho nikdo nepustil. Recenzent si typy „ověřil čtením“.

Nebo si všimne, že test na novějším PHP zčervená, a napíše: *„To je zamýšlená pojistka do budoucna.“* Do konfigurace CI se nepodívá. Škoda. Viděl by, že ta budoucnost v CI běží už dnes.

Recenzent vidí diff. Zbytek systému si musí dohledat sám. A když ho k tomu nic nenutí, nedohledá. Proto ať nástroje pouští sám: statickou analýzu, testy, všechny podporované verze.

Recenzent narazí na něco podezřelého a mávne nad tím rukou: *„to je přijatelné“*, *„to je záměr“*, *„to se v praxi nestane“*. Občas má pravdu. Jenže tudy chyby unikají. Níž je případ, kdy to recenzent udělal třikrát. Takové odbytí ať napíše, jen když ukáže, kde se to projeví: kdo tu metodu volá, odkud do ní tečou data, v jakém prostředí běží. Když to dohledat nejde, není to „přijatelné“, je to otevřená otázka.

Starší chyby, které recenze propustila, jsem nechal zrecenzovat znovu s různými zadáními, každé zadání třikrát:

- **Chyba, která se projeví jen na novější verzi PHP.** Ani s původním zadáním, ani s *„buď skeptický“*, ani s *„měř“* ji recenzent jako vadu nenahlásil ani jednou. Třikrát ji zahlédl a odbyl přesně tou větou o pojistce. Chytilo ji až zadání *„měř a dohledej, kde se to projeví“*, a to **ve všech třech bězích**: recenzent otevřel konfiguraci CI a změřil to.
- **Okrajový vstup, který projde validací a knihovna na něm spadne.** Tady stačilo měřit. Se zadáním *„měř“* ho recenzent nahlásil jako vadu ve dvou bězích ze tří. Bez měření ani jednou, nanejvýš ho zmínil jako hypotézu.

Tři běhy na variantu nejsou statistika, jen směr. Ten je ale jasný: „buď skeptický“ skoro nic nezmění.

**Skepse bez nástroje je jen přísnější čtení.**

## 3. Druhý názor, který jím není

Agent dopíše úpravu a hlásí: *„Tlačítko Uložit teď uvidí jen správce.“* Druhý agent to má zkontrolovat. Otevře šablonu. Vidí podmínku na roli, vidí obsluhu kliknutí. Souhlasí. A na stránce tlačítko nefunguje nikomu, protože skript o řádek výš spadne na chybě. Nikdo to neviděl, protože se nikdo nepodíval.

Dvě čtení, jeden zdroj. Jedno ověření se dvěma podpisy.

Ověření musí přijít odjinud než tvrzení. Co vidí uživatel, dokládá vykreslená stránka: konzole, DOM, screenshot. Jak se kód chová, dokládá test nebo měření. A když uživatel hlásí něco jiného, než říká kód? Platí hlášení, ne kód. Rozporovat ho můžete, až si to sami zopakujete.

**Dvakrát přečteno není dvakrát ověřeno.**

## 4. „Prošlo“, které nic nedokazuje

Agent hlásí: *„Testy prochází, lint je čistý.“* Prošly. Před třemi úpravami. Nebo pustil jeden nástroj a ohlásil, že prošla celá sada. Nebo zpráva vyjmenuje, co prošlo, a o tom, co se vůbec nepustilo, mlčí.

Zpráva o stavu vypadá jako fakt a nikdo se nezeptá, z jakého běhu je. Proto ať vždycky obsahuje tři věci: jaký příkaz agent pustil po poslední úpravě, co ten příkaz vypsal na posledním řádku a co nepustil vůbec. Ještě líp v pevném tvaru, třeba takhle:

```text
type=report  task=3  commit=a1b2c3d
measured:  composer test → OK (120 tests, 342 assertions)
not run:   E2E tests – no database
```

`commit` říká, nad jakým stavem kódu se měřilo. Když se liší od posledního commitu, hlášení je prošlé. A pevný tvar jde hlídat kódem: hlášení, kterému chybí `measured` nebo `not run`, se vrátí zpátky.

A když `not run` chybí nebo je prázdné? Neznamená to, že se pustilo všechno. Znamená to, že se něco zamlčelo.

**Hlášení bez výpisu je jen dojem.**

## 5. Domněnka, která se tváří jako fakt

API vrací `403` a agent neví proč. Tak si to domyslí. Do shrnutí i do komentáře v kódu napíše: *„Endpoint vyžaduje roli správce.“* Jako fakt. Nikdo to neřekl, nikde to nestojí. Další agent si to přečte v komentáři a od té chvíle je to pro něj pravda.

Domněnka napsaná sebejistě vypadá jako zjištění. Co agent jen usuzuje, patří mezi otázky pro člověka, ne do textu jako tvrzení. S čísly totéž: buď změřené, nebo označené jako odhad.

**Znít jistě není totéž co vědět.**

## Jedna vrstva nestačí

Zpětně jsem prošel 18 prací a spočítal vážné vady (chyby, bezpečnostní problémy, mezery v testech), které se v nich našly: 150. **35 z nich proklouzlo recenzí tasku** a chytila je až další vrstva, nejčastěji finální review celé větve. Zbylých 115 chytila recenze tasku. Funguje tedy, jen nestačí sama. Vzorek je malý a část závažností jsem určoval zpětně, takže čísla berte orientačně.

## Co tohle neřeší

Žádná pojistka není zadarmo ani úplná: když se ptá pořád, lidi se naučí odklikávat, a co jde udělat jinou cestou, to projde. Proti obcházení pomáhá sandbox: přístup k síti a disku pak hlídá operační systém, ať agent sáhne po jakémkoli nástroji. I on ale pustí všechno, co mu povolíte. Každá další kontrola navíc stojí čas i tokeny.

## Checklist

- Co agent nesmí udělat sám (zápis ven, push, produkce), hlídá **hook nebo oprávnění**, ne věta v promptu.
- Recenzent **pouští analýzu a testy sám**. Co nepustil, je mezera, ne „prošlo“.
- Nález smí recenzent odbýt jen tehdy, když **ukáže, kde se projeví**. Jinak je to otevřená otázka.
- Ověření přichází **odjinud než tvrzení**: co vidí uživatel, z vykreslené stránky, ne ze zdrojáku.
- Stav se hlásí jako **příkaz po poslední úpravě, poslední řádek a co se nepustilo**.
- Domněnka agenta je **otázka**, ne fakt.
- Chyby, které kontrola propustila, si **zapisujte**. Jsou to nejlepší testy pro příští zadání recenzenta.

**Prompt říká, co se má stát. Jestli se to opravdu stalo, musí ověřit něco jiného: kód, měření, druhý zdroj, člověk.**
