---
layout: post
title: "🔥 fetchAll() v DataGridu? Právě jste zabili server — aneb proč Selection není pole"
description: "Předčasné stahování dat přes fetchAll() dokáže spolehlivě zabít server. Naučte se, jak v Nette DataGridu filtrovat chytře pomocí SQL receptů."
date: 2026-05-16 20:15:00 +0200
categories: [Programming, PHP]
tags: [php, nette, database, optimization, datagrid, contributte]
image:
  path: assets/img/posts/2026-05-16-nette-selection-database.png
---

Už se vám to někdy stalo? Píšete datagrid, filtry pro administraci nebo export dat. Vytvoříte dotaz přes `Nette\Database`, chcete ho předat do gridu, ale v dobré víře napíšete tohle:

```php
// ❌ ŠPATNĚ: Tímto do gridu předáte natvrdo pole hotových řádků
$orders = $this->database->table('order')->fetchAll();
$grid->setDataSource($orders);
```

Všechno funguje skvěle... dokud na produkci nepřeteče webová paměť, neuvaříte procesor a uživatel neuvidí jen strohou chybu **500 Internal Server Error**.

Pojďme se podívat na to, proč k tomu dochází a proč je klíčem k úspěchu pochopit, že správný zápis vypadá takto a že `Nette\Database\Table\Selection` není pole dat, ale **recept**:

```php
// ✅ SPRÁVNĚ: Gridu předáváte pouze "recept", data si vytáhne sám až na konci
$orders = $this->database->table('order');
$grid->setDataSource($orders);
```

## Recept vs. Hotové jídlo

Když v Nette zavoláte `$orders = $this->database->table('order');`, v proměnné `$orders` **nemáte** tisíce řádků s objednávkami z databáze. Nemáte tam ani jeden.

`Selection` je v podstatě jen "objednávkový lístek" nebo recept pro databázi. Říkáte tím: *"Až po tobě budu něco chtít, připrav se, že půjdeme do tabulky objednávek."*

Když k tomu přidáte podmínku:
```php
$orders->where('status', 'paid');
```
Stále se z databáze nic nestáhlo. Jen jste na ten objednávkový lístek připsali: *"...a chci jen ty zaplacené."*

Všimněte si, že gridu přes `$grid->setDataSource($orders)` nepředáváte pole hotových dat, ale pořád jen ten **recept**. Grid si do něj v průběhu životního cyklu sám přidá další ingredience – například podle toho, co uživatel zrovna vyhledává ve filtrech, nebo na jakou stránku kliknul (přidá si vlastní `LIMIT` a `OFFSET`).

K databázi se dotaz fyzicky odešle až na úplném konci, kdy se z `Selection` pokusí data poprvé skutečně vytáhnout pro vykreslení tabulky.

## Vražda serveru v přímém přenosu (Anti-pattern)

Představte si, že implementujete vyhledávání v administraci objednávek. Chcete filtrovat podle jména zákazníka. Protože ale máte různé typy objednávek (např. pro firmy a koncové uživatele) a jména jsou v různých tabulkách, rozhodnete se to vyřešit "pohodlně" v PHP:

```php
$grid->addFilterText('customerName', 'Jméno zákazníka')
    ->setCondition(function (Selection $selection, string $value) {
        // Tímto přinutíte Nette, aby DOOPRAVDY stáhlo VŠECHNY objednávky do PHP paměti
        $allOrders = $selection->fetchAll(); 

        $filteredIds = [];
        foreach ($allOrders as $order) {
            // Načteme složitou entitu a filtrování děláme v PHP
            $customerName = $order->ref('customer')->name;
            if (str_contains(mb_strtolower($customerName), mb_strtolower($value))) {
                $filteredIds[] = $order->id;
            }
        }

        // Vrátíme vyfiltrovaná ID zpět do původní selection
        $selection->where('id', $filteredIds);
    });
```

### Proč je to cesta do pekla?
Pokud máte v e-shopu 500 objednávek, aplikace bude bleskově rychlá. Pokud jich tam ale budete mít 50 000, stane se následující:
1. **PHP Memory Limit:** Vykouzlíte SQL dotaz `SELECT * FROM order`, který do RAM paměti PHP serveru nahrne obrovský balík dat. PHP dojde paměť a skript okamžitě padá na `Fatal error: Allowed memory size exhausted`.
2. **Zbytečná režie procesoru:** CPU serveru bude v cyklu `foreach` zběsile porovnávat tisíce řetězců, což zabere sekundy. Uživatel mezitím kouká na načítající se kolečko.
3. **Zmařená práce:** Všechna ta data jste stáhli jen proto, abyste z nich vytáhli pár IDček a databáze se za chvíli zeptali znovu na to samé.

## Správná cesta: Nechte pracovat databázi

Databáze (ať už MySQL, PostgreSQL nebo jiná) jsou neuvěřitelně optimalizované stroje. Jsou stavěné na to, aby bleskově prohledávaly miliony řádků, spojovaly tabulky a filtrovaly data. Vy jako programátoři jim k tomu musíte dát jen ten správný **recept**.

Místo toho, abychom data tahali do PHP a tam je pracně prosívali, musíme upravit samotný SQL dotaz uvnitř `Selection`.

```php
$grid->addFilterText('customerName', 'Jméno zákazníka')
    ->setCondition(function (Selection $selection, string $value) {
        // Pouze upravíme "recept" (SQL dotaz) pomocí EXISTS a parametrů
        $selection->where("
            EXISTS (
                SELECT 1 FROM `customer` `c` 
                WHERE `c`.`id` = `order`.`customer_id` 
                AND `c`.`name` LIKE ?
            )
        ", "%$value%");
    });
```

### Co se stalo teď?
Do paměti PHP se nenačetl **ani jeden řádek navíc**. Nette gridu nepředalo data, ale upravený *recept*. Datagrid si k tomuto receptu na konec přihodí ještě `LIMIT 20 OFFSET 0` (pro stránkování) a až TEĎ pošle dotaz do databáze.

Databáze prohledá své indexy, bleskově vybere přesně těch 20 konkrétních řádků, které chce uživatel vidět na první stránce, a jen těch 20 řádků doputuje do PHP. Výsledek? **Zpracování trvá 0,02 sekundy a spotřeba paměti je minimální.**

## Shrnutí pro váš příští kód

Kdykoliv pracujete se `Selection` v Nette, držte se pravidla: **PHP data pouze zobrazuje, databáze je filtruje a třídí.**

* Pokud v komponentě (jako je grid) filtrujete data, **nikdy** nepoužívejte `fetchAll()` nebo cykly nad celou tabulkou.
* Využívejte sílu SQL (`WHERE`, `JOIN`, `EXISTS`).
* Předávejte hodnoty přes parametry (`?`), Nette se postará o bezpečnost (SQL Injection) a správné uvozování řetězců.

Vaše servery vám poděkují a vaši uživatelé neuvidí chybu 500, ale okamžitě načtenou stránku.
