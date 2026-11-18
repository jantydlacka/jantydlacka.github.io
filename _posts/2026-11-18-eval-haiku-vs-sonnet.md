---
layout: post
lang: cs
page_id: eval-haiku-vs-sonnet
title: "📏 Haiku, nebo Sonnet? Neodhaduj to, změř to"
description: "Haiku proti Sonnetu na vytahování údajů z dokumentů: 79 z 80 prošlo, takže nevyhrál nikdo. Zato vyšlo najevo, kolik stojí přemýšlení a že i dobrý model umí vrátit prázdnou odpověď."
date: 2026-11-18 08:00:00 +0100
categories: [Programming, AI]
tags: [ai, llm, evals, claude, costs]
image:
  path: /assets/img/posts/2026-11-18-eval-haiku-vs-sonnet.png
---

Levnější model, nebo chytřejší model? Na tuhle otázku má každý názor. Obvykle podložený třemi
pokusy v chatu a dobrým pocitem.

Pocit se ale do tabulky nezapíše. Tak jsem to změřil.

Spoiler: nevyhrál nikdo. A právě proto to stojí za článek.

## Úloha

Z dokumentu vytáhnout šest údajů do JSONu: číslo smlouvy, ISIN, jestli má ISIN platnou
kontrolní číslici, částku, měnu a jestli je dokument vůbec čitelný.

Dvacet vymyšlených dokumentů – smlouvy, výpisy, potvrzení pokynů. Šestnáct textových PDF, čtyři obrázkové skeny.
A schválně pár podrazů:

- ISIN vytištěný po skupinách s mezerami,
- ISIN s chybnou kontrolní číslicí,
- tři částky v dokumentu, z nichž platí jedna,
- chybějící číslo smlouvy,
- natočený a zašuměný sken,
- sken tak zničený, že jediná správná odpověď je „tohle se přečíst nedá“.

Proti sobě stojí Claude Haiku 5.5 a Claude Sonnet 5.5, oba s `effort: medium`. Každý dostal dvě verze
promptu: v1 stručná, v2 s pravidly pro záludné případy.

## Známkuje kód, ne další model

Žádný LLM jako soudce. Když je výstup JSON, porovná ho obyčejný kód. Pole po poli, bez
shovívavosti:

- ISIN s mezerami? Chyba formátu.
- `""` místo `null`? Taky chyba. Navazující kód pozná chybějící údaj podle `null` – prázdný řetězec by vzal jako hodnotu a uložil prázdné číslo smlouvy.
- Odpověď useknutá v půlce? Neprošlo, i kdyby to, co stihla, sedělo.

A dvě kontroly samotného měřítka. Falešný model, který opisuje správné odpovědi, musí projít
všech 20 případů – jinak je chyba v graderu, ne v modelu. Falešný model, který dokument vůbec
neotevře a na všechno odpoví „nic tu není“, nesmí projít ani jeden. Kdyby některý případ prošel,
dal by se splnit bez čtení dokumentu.

## Peníze hlídá kód, ne svědomí

Eval, který tiše utratí víc, než měl, je špatný eval.

- Před během nástroj spočítá horní mez ceny. Vyšla na 2,01 $.
- Nad 1 $ chce výslovné potvrzení částky. Kdyby celková útrata přesáhla 5 $, odmítne začít.
- Každé volání se zapíše do evidence útraty. Volání, na které nepřišla odpověď, dostane značku „nevíme, jestli se účtovalo“ a do stropu se počítá nejhorší možnou cenou, ne nulou.

Skutečná útrata za hlavní běh: **0,47 $**, se třemi zkušebními voláními předem 0,53 $. Horní mez je schválně pesimistická – počítá s tím, že
každá odpověď vyčerpá celý limit tokenů.

## Výsledek: remíza

| | Haiku v1 | Haiku v2 | Sonnet v1 | Sonnet v2 |
|---|---|---|---|---|
| prošlo | 20/20 | 19/20 | 20/20 | 20/20 |
| cena za úspěšný případ | 0,00065 $ | 0,00073 $ | 0,0106 $ | 0,0115 $ |
| výstupních tokenů na případ | 745 | 732 | 494 | 495 |
| medián latence | 4,33 s | 4,51 s | 4,66 s | 4,53 s |

79 z 80. A rychlostí se modely skoro neliší – medián kolem 4,5 sekundy u obou.

Gratuluju, máme sadu, která neodliší dva modely ani dva prompty. Je prostě moc snadná.

Tohle není neúspěch. Tohle je přesně ta informace, kterou eval má dát: **tahle data na to
rozhodnutí nestačí.** Kdo by z 20/20 vyvodil „Haiku je stejně dobrý jako Sonnet“, vyvozuje
z testu, který nikoho nenechal propadnout.

## Co ale vyšlo najevo

### Dvacetkrát dražší? Šestnáctkrát.

Sonnet má v ceníku dvacetkrát vyšší sazby než Haiku, na vstupu i na výstupu. Změřená cena je
ale vyšší jen zhruba 16×.

Proč? Haiku víc přemýšlí. Na případ vyprodukoval průměrně 745 výstupních tokenů, Sonnet 494.
Přemýšlení z toho tvořilo 91 % u Haiku a 87 % u Sonnetu. Levnější model si část slevy prostě
propřemýšlí.

Ceník říká cenu za token. Kolik tokenů model spotřebuje, neříká. To musíte změřit.

Mimochodem: parametr `thinking` nástroj vůbec neposílá. Modely přesto přemýšlely v 78 z 83 volání (80 v běhu a 3 zkušební). Do odhadu nákladů s tím počítejte.

Vstup se naopak dá odhadnout přesně: bezplatné počítání tokenů v API (`count_tokens`) se ve všech třech zkušebních voláních shodovalo s fakturovaným vstupem přesně na token. Nejistota ceny je celá ve výstupu.

### Delší prompt se platí každým voláním

Pravidla navíc ve v2 přidala 492 vstupních tokenů na volání. Úspěšnost nezvedla – sada byla u
stropu už s v1. Cenu zvedla o 6,6 % u Haiku a o 9,4 % u Sonnetu.

Každá věta v promptu je předplatné.

### Jediná chyba je ta nejpoučnější

Haiku s promptem v2 u platného ISIN tvrdil, že kontrolní číslice nesedí.

Kontrolní číslice ISIN se počítá pevným algoritmem. Pár řádků kódu, pokaždé stejný výsledek.
Podle mě ji model vůbec počítat nemá – ať vytáhne ISIN a platnost spočítá kód. Věci, které jdou spočítat
deterministicky, nemají být v promptu.

### I dobrý model umí vrátit nic

Při zkušebním volání před během dostal Sonnet natočený sken. Přemýšlel. A přemýšlel. Spotřeboval
celý limit 4 096 tokenů, z toho 4 095 na přemýšlení – a vrátil prázdnou odpověď. 4 096
výstupních tokenů naúčtovaných za nic.

V plném běhu prošel stejný případ s 520 tokeny bez problému.

Jak často se to stává, z jednoho výskytu nevyčtete. Jen to, že se to stát může. Takže: limit
tokenů je součást podmínek měření, useknutá odpověď je selhání a nástroj ji musí vypsat zvlášť.

## Co tenhle eval neříká

- Data jsou syntetická, z jednoho generátoru, jedním písmem. O reálných skenech toho moc neřekne.
- Dvacet případů, jedno opakování. Výstup modelu není deterministický – rozdíl jednoho případu je šum.
- `effort: medium` je stejný jen nominálně. Úrovně effortu nejsou mezi modely sjednocené.
- Ceny platí k 16. 11. 2026.

## Co si odnést

- Nerozhodujte podle pocitu ani podle cizího benchmarku. Rozhodujte na vlastních datech.
- Když všichni projdou, nemáte vítěze. Máte moc snadný test.
- Známkujte kódem všude, kde to jde. Sporné případy rozhodněte předem.
- Cenu čtěte z odpovědi API, ne z ceníku. Přemýšlení je položka na faktuře.
- Útratu hlídejte kódem.
- Co jde spočítat, ať počítá kód, ne model.

Další krok je jasný: těžší sada. Reálnější skeny, víc šablon, víc opakování. Teprve pak se
ukáže, jestli se Sonnet vyplatí.

Kód, data a surové výsledky: [github.com/jantydlacka/doc-extraction-eval](https://github.com/jantydlacka/doc-extraction-eval) (shrnutí v [docs/vysledky.md](https://github.com/jantydlacka/doc-extraction-eval/blob/main/docs/vysledky.md)).

Ke čtení:
- [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) – Hamel Husain
- [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) – Anthropic
- [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) – ceník Claude
