---
layout: post
title: "🔓 Jak zrušit trvalé přesměrování na HTTPS v prohlížeči"
description: "Rychlý návod, jak vyřešit problém s nechtěným přesměrováním na HTTPS na localhostu v prohlížečích Brave a Chrome."
date: 2026-03-25 14:00:00 +0100
categories: [Programming, Web]
tags: [localhost, https, devtools, hsts, brave, chrome]
image:
  path: assets/img/posts/2026-03-25-https-redirect-browser.png
---

Stane se to i v lepších rodinách. Ladíte `.htaccess` na localhostu, nastavíte si „na zkoušku“ přesměrování na HTTPS přes kód **301 (Permanent)**, a pak zjistíte, že i když pravidlo smažete, prohlížeč vás na HTTP už prostě nepustí.

Zde je rychlý manuál, jak z toho ven.

### 1. První pomoc: DevTools a „Disable Cache“

Tohle je nejrychlejší cesta, pokud potřebujete okamžitě vidět změnu a neřešit hloubkovou očistu prohlížeče.

1. Otevřete **Vývojářské nástroje** (`F12`).
2. Přejděte na kartu **Network** (Sítě).
3. Zaškrtněte políčko **Disable cache**.

Dokud je panel `F12` otevřený, prohlížeč bude ignorovat uložená přesměrování a načte web „na čisto“ ze serveru.

### 2. Tvrdý restart (Hard Reload)

Pokud chcete, aby to fungovalo i po zavření vývojářských nástrojů, musíte mezipaměť pro danou stránku vyloženě „vykopnout“:

1. Mějte otevřené **DevTools** (`F12`).
2. Klikněte pravým tlačítkem na ikonu pro obnovení stránky (kulatá šipka v adresním řádku).
3. Zvolte **Vyprázdnit mezipaměť a vynutit opětovné načtení** (Empty Cache and Hard Reload).

### 3. Deep Clean: Smazání HSTS domény

Někdy prohlížeč získá pocit, že localhost patří do seznamu bezpečných webů (HSTS) a odmítá se bavit o čemkoliv jiném než HTTPS.

1. Do adresního řádku Brave/Chrome vložte: `brave://net-internals/#hsts` (nebo `chrome://net-internals/#hsts`).
2. Sjeďte dolů k sekci **Delete domain security policies**.
3. Do pole **Domain** napište: `localhost` a potvrďte tlačítkem **Delete**.

### Proč se to stalo? (Lekce pro příště)

Chyba byla v použití kódu **301**. Ten říká prohlížeči: „Tenhle web se trvale přestěhoval, už se sem pro HTTP nikdy nevracej.“ Prohlížeč si to zapíše do databáze na disk a příště už se serveru ani neptá.

> **Zlaté pravidlo vývojáře:**
>
> Při testování na localhostu používejte v `.htaccess` zásadně kód **302 (Found/Temporary)**. Ten prohlížečům říká, aby se příště znovu zeptaly, a vy tak zůstanete pánem situace.
