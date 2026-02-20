---
layout: post
title: "⏰Proč mi Chronos krade čas? Pozor na update 3.1.0"
description: "Zjistěte, proč aktualizace knihovny Chronos na verzi 3.1.0 může posunout vaše data v DB o den zpět a jak tento timezone bug elegantně opravit."
date: 2026-02-20 21:00:00 +0100
categories: [Programming, PHP]
tags: [chronos, nette, bug, timezone]
image:
  path: /assets/img/posts/chronos-thief.jpg  # Cesta k tvému obrázku
  alt: Timezone confusion diagram
---

Při vývoji v Nette (nebo jakémkoliv jiném PHP frameworku) často spoléháme na to, že "čas je prostě čas". Jenže pak přijde nenápadný update knihovny a najednou zjistíte, že se vám data v databázi posouvají o den dozadu.

### Problém: Půlnoc, která není půlnocí

Představte si, že máte v databázi uloženo datum `2026-02-04 00:00:00`. Na webu se ale uživateli zobrazí `2026-02-03`. Kde zmizela ta hodina?

Může za to kombinace **Unix Timestampu** a změny v knihovně **Cake\Chronos 3.1.0**.

### Zrádný Timestamp

Kód, který dříve fungoval, vypadal takto:

```php
return Chronos::createFromTimestamp($row->datum->getTimestamp());

```

**Co se děje uvnitř:**

1. `$row->datum` je objekt (např. `Nette\Utils\DateTime`) v zóně `Europe/Prague`.
2. Metoda `getTimestamp()` převede tento čas na vteřiny od roku 1970. **Timestamp je ale vždy v UTC.**
3. Půlnoc v Praze (v zimě) znamená **23:00 předešlého dne v UTC**.

### Změna v Chronos 3.1.0 (Příprava na PHP 8.4)

Dříve se Chronos při vytváření z timestampu pokusil "hádat" zónu podle nastavení serveru. Od verze 3.1.0 ale došlo k zásadní změně:

> Pokud není časová zóna výslovně předána, metoda `createFromTimestamp()` nyní natvrdo vrací objekt v **UTC (+00:00)**.

Tím pádem získáte objekt s časem `23:00:00 UTC`. Jakmile na něj zavoláte `->format('Y-m-d')`, vypíše se včerejší datum.

### Jak z toho ven?

#### 1. Elegantní cesta (Doporučeno)

Pokud už máte k dispozici objekt `DateTime` (což Nette Database vrací automaticky), nepoužívejte timestamp jako prostředníka. Použijte metodu `instance()`:

```php
// Chronos si z původního objektu vytáhne čas i správnou zónu (Prahu)
return Chronos::instance($row->datum);

```

#### 2. Explicitní cesta

Pokud máte skutečně jen timestamp (číslo), musíte Chronosu říct, v jaké zóně ho chcete interpretovat:

```php
return Chronos::createFromTimestamp($ts, 'Europe/Prague');

```

### Ponaučení pro příště

* **Timestampy jsou zrádné:** Při převodu mezi lokálním časem a timestampem vždy hrozí ztráta informace o časové zóně.
* **Čtěte changelogy:** I minoritní update (3.0 -> 3.1) může obsahovat změnu chování (tzv. behavior change), která rozbije existující logiku.
* **PHP 8.4:** Tato změna v Chronosu není náhodná – připravuje nás na standardizaci, která přichází přímo do jádra PHP.

<style>
  /* Oprava fontu pro českou diakritiku v detailu článku */
  .post-desc {
    font-style: normal !important;
    font-weight: 400 !important; /* Nastaví normální tloušťku, která česky umí */
  }
</style>
