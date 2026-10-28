---
layout: post
lang: cs
page_id: php-strict-types
title: "⚠️ Kde vás strict_types=1 nechrání"
description: "strict_types=1 nehlídá všechno. Návratové typy řídí jiný soubor než argumenty, callbacky z array_map se přetypují dál a (int) nezkontroluje vůbec nic."
date: 2026-05-29 08:00:00 +0200
categories: [Programming, PHP]
tags: [php, strict-types, type-system, best-practices]
image:
  path: /assets/img/posts/2026-05-29-php-strict-types.png
---

Píšete `declare(strict_types=1)` na začátek každého PHP souboru. IDE vám to přidá automaticky. Ale co to přesně dělá? A kdy může překvapit na místě, kde to nečekáte?

## Jak PHP funguje bez strict_types

PHP je od přírody shovívavý jazyk. Pokud funkce očekává `int` a vy jí pošlete `"5"`, PHP krčí rameny, tiše přetypuje a pokračuje dál:

```php
function calculateDiscount(int $percent, int $price): int
{
    return (int) ($price * $percent / 100);
}

echo calculateDiscount("20", "1000"); // 200 — funguje, žádná chyba
```

Toto „shovívavé“ chování se jmenuje **coercion** (nucené přetypování). Obvykle se děje tiše a ve většině případů to dopadne dobře. Ve zbytku případů dostanete záhadný bug, který se projeví až za týden.

## Co strict_types zapne

Jakmile na začátek souboru napíšete `declare(strict_types=1)`, PHP se přestane chovat shovívavě. Předání špatného typu napříště není tichá konverze — je to `TypeError`:

```php
declare(strict_types=1);

echo calculateDiscount("20", "1000");
// TypeError: Argument #1 ($percent) must be of type int, string given
```

Chyba se projeví okamžitě a hlasitě, místo aby se tiše šířila dál a projevila někde úplně jinde.

## Největší překvapení: strict_types je vlastnost VOLAJÍCÍHO souboru

Tady začíná věc, kvůli které píšu tento post. Často se má za to, že `declare(strict_types=1)` je vlastnost *funkce* nebo *třídy*. Není.

Je to vlastnost **souboru, ze kterého funkci voláte**.

Mějme funkci definovanou v souboru bez strict_types:

```php
// math.php — definice funkce BEZ strict_types
function add(int $a, int $b): int
{
    return $a + $b;
}
```

Teď ji zavoláme ze dvou různých souborů:

```php
// caller_strict.php — SE strict_types
declare(strict_types=1);

require 'math.php';
add("5", "3");   // 💥 TypeError — přísnost volajícího rozhoduje
add("5.9", "3"); // 💥 TypeError
```

```php
// caller_loose.php — BEZ strict_types
require 'math.php';
add("5", "3");   // 8 — tiché přetypování, žádná chyba
add("5.9", "3"); // 8 — desetinná část zmizela, PHP jen zahlásí Deprecated
```

Stejná funkce, stejný kód, stejné argumenty — a přitom jiný výsledek podle toho, ze *kterého souboru* voláte.

Pokud to vidíte poprvé, pravděpodobně vás to překvapí stejně jako mě.

Druhý řádek je přesně ten záhadný bug z úvodu: v souboru bez strict_types je výsledek špatně a jedinou stopou je hláška Deprecated, kterou produkční prostředí často nezobrazuje ani neloguje. Strict soubor stejné volání zastaví TypeErrorem.

Pozor ale, platí to jen pro **argumenty**. Návratovou hodnotu kontroluje soubor, kde je funkce *definovaná*. Funkce ze strict souboru, která vrátí `"7"` místo `int`, hodí TypeError i tehdy, když ji voláte ze souboru bez strict_types.

## Kde strict_types neplatí

Přísnost se týká předávání hodnot do typovaných parametrů, návratových hodnot a typovaných properties. Jinde PHP přetypovává dál:

- **int → float projde.** Funkce s parametrem `float` přijme `5` i ve strict režimu a dostane `5.0`.
- **Operátory a porovnání.** `"5" + 3` je pořád `8` a `"0" == false` pořád `true`.
- **Callbacky volané z vestavěných funkcí.** Tady je hranice nejméně vidět:

```php
declare(strict_types=1);

namespace App;

$toInt = fn(int $x): int => $x;

$toInt("5");                  // 💥 TypeError
\call_user_func($toInt, "5"); // 💥 TypeError
call_user_func($toInt, "5");  // 5 — tiché přetypování
array_map($toInt, ["5"]);     // [5] — tiché přetypování
```

Callback z `array_map`, `array_filter` nebo `usort` nevoláte vy, ale vestavěná funkce PHP. A ta žádné `strict_types` nemá. Zrádný je `call_user_func`: v souboru s namespacem — tedy v každé běžné třídě — se bez úvodního `\` chová stejně jako `array_map`. TypeError dostanete jen s `\call_user_func` (nebo s `use function call_user_func;`), případně v souboru bez namespacu.

## Praktický dopad: z formuláře nikdy nepřijde int

HTTP formulář posílá **text**. Nezáleží na tom, jaký `type="number"` máte v HTML. Co přijde do PHP přes `$_POST` nebo `$request->getPost()`, je string. Případně pole (u `name="items[]"`), nebo `null`, když hodnota ve formuláři chybí.

V Nette presenterech máte `declare(strict_types=1)` (a správně), takže tohle hodí TypeError:

```php
declare(strict_types=1);

// $request->getPost('quantity') vrátí "5", ne 5
$this->orderService->setQuantity(
    $request->getPost('quantity') // 💥 TypeError
);
```

Nabízí se explicitní přetypování. Jenže `(int)` nic nekontroluje:

```php
// "abc" → 0, null → 0, bez chyby
$quantity = (int) $request->getPost('quantity');
```

Paradoxně je tak volnější než samo tiché přetypování: `"abc"` předané do parametru `int` hodí TypeError i bez strict_types. Když chcete neplatný vstup odmítnout, použijte validaci nebo Nette Forms:

```php
// Validace — neplatný vstup (i null nebo pole) vrátí false
$quantity = filter_var(
    $request->getPost('quantity'),
    FILTER_VALIDATE_INT,
);
if ($quantity === false) {
    // chybná hodnota
}

// Nebo Nette Forms — addInteger() hodnotu zvaliduje a převede
$form->addInteger('quantity');
// int, u prázdného pole null ✅
$quantity = $form->getValues()->quantity;
```

## Co strict_types neřeší: falsy "0"

`strict_types` vás ochrání před tichým přetypováním argumentů, ale podmínky, `?:` ani `empty()` vůbec neovlivňuje.

Klasická past: výchozí hodnota pro nevyplněné pole.

```php
declare(strict_types=1);

// „Když uživatel nic nezadá, dej výchozích 10 %“
$discount = (int) ($request->getPost('discount') ?: 10);
```

Uživatel zadá slevu 0, ale dostane 10 %. Řetězec `"0"` je v PHP falsy stejně jako `""`, `null`, `0` nebo `[]`, takže ho `?:` přeskočí. Stejně dopadne `empty()`, které nerozliší nulu od nevyplněného pole.

Pokud je nula platná hodnota, rozlište „nevyplněno“ explicitně:

```php
$raw = $request->getPost('discount');
$discount = ($raw === null || $raw === '') ? 10 : (int) $raw;
```

## Shrnutí

- `strict_types=1` zakáže tiché přetypování při předávání argumentů, vracení hodnot a zápisu do typovaných properties — záhadné bugy se projeví jako hlasitý TypeError
- U argumentů rozhoduje **soubor volajícího**, u návratových hodnot soubor, kde je funkce definovaná
- Neplatí pro int → float, operátory, porovnání a callbacky volané z vestavěných funkcí (`array_map` a spol., v namespacu i `call_user_func`)
- Z formuláře přijde string, pole nebo null — `(int)` nic nevaliduje, použijte `filter_var` nebo Nette Forms
- strict_types je pojistka, ne náhrada za validaci — logické chyby (falsy `"0"`, záporné číslo tam, kde ho nechcete) musíte řešit sami

Platí tedy jednoduché pravidlo: `declare(strict_types=1)` patří do každého souboru. Ale vědomost o tom, *jak přesně* funguje, vám ušetří dost zmateného debugování.
