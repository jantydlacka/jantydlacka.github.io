---
layout: post
lang: en
page_id: php-strict-types
title: "⚠️ Where strict_types=1 Won't Protect You"
description: "strict_types=1 doesn't guard everything. Return types follow a different file than arguments, array_map callbacks still coerce, and (int) checks nothing at all."
date: 2026-05-29 08:00:00 +0200
categories: [Programming, PHP]
tags: [php, strict-types, type-system, best-practices]
image:
  path: assets/img/posts/2026-05-29-php-strict-types-en.png
permalink: /posts/php-strict-types/
---

You write `declare(strict_types=1)` at the top of every PHP file. Your IDE adds it automatically. But what does it actually do? And when can it catch you off guard?

## How PHP Works Without strict_types

PHP is a forgiving language by nature. If a function expects an `int` and you pass `"5"`, PHP shrugs, silently converts it, and carries on:

```php
function calculateDiscount(int $percent, int $price): int
{
    return (int) ($price * $percent / 100);
}

echo calculateDiscount("20", "1000"); // 200 — works, no error
```

This "forgiving" behavior is called **coercion** (forced type conversion). It usually happens silently, and in most cases it works out fine. In the remaining cases, you get a mysterious bug that surfaces somewhere completely unexpected a week later.

## What strict_types Turns On

As soon as you add `declare(strict_types=1)` at the top of a file, PHP stops being forgiving in that file. Passing the wrong type is no longer a silent conversion — it's a `TypeError`:

```php
declare(strict_types=1);

echo calculateDiscount("20", "1000");
// TypeError: Argument #1 ($percent) must be of type int, string given
```

The error appears immediately and loudly, instead of silently spreading and showing up somewhere completely different.

## The Biggest Surprise: strict_types Is a Property of the CALLING File

Here's the part this post is really about. It's often assumed that `declare(strict_types=1)` is a property of the *function* or *class*. It isn't.

It's a property of **the file you're calling the function from**.

Consider a function defined in a file without strict_types:

```php
// math.php — function defined WITHOUT strict_types
function add(int $a, int $b): int
{
    return $a + $b;
}
```

Now call it from two different files:

```php
// caller_strict.php — WITH strict_types
declare(strict_types=1);

require 'math.php';
add("5", "3");   // 💥 TypeError — the caller's strictness decides
add("5.9", "3"); // 💥 TypeError
```

```php
// caller_loose.php — WITHOUT strict_types
require 'math.php';
add("5", "3");   // 8 — silent coercion, no error
add("5.9", "3"); // 8 — decimals gone, PHP only emits a Deprecated
```

Same function, same code, same arguments — and yet different behavior depending on *which file* is calling it.

If you're seeing this for the first time, you're probably just as surprised as I was.

The second line is exactly the mysterious bug from the intro: in the file without strict_types the result is wrong, and the only trace is a Deprecated notice, which production setups often neither display nor log. The strict file stops the same call with a TypeError.

But careful: this applies to **arguments** only. Return values are checked by the file where the function is *defined*. A function from a strict file that returns `"7"` instead of an `int` throws a TypeError even when you call it from a file without strict_types.

## Where strict_types Doesn't Apply

Strictness covers passing values into typed parameters, return values and typed properties. Everywhere else PHP keeps coercing:

- **int → float is allowed.** A function with a `float` parameter accepts `5` even in strict mode and receives `5.0`.
- **Operators and comparisons.** `"5" + 3` is still `8` and `"0" == false` is still `true`.
- **Callbacks invoked by built-in functions.** This is where the line is least visible:

```php
declare(strict_types=1);

namespace App;

$toInt = fn(int $x): int => $x;

$toInt("5");                  // 💥 TypeError
\call_user_func($toInt, "5"); // 💥 TypeError
call_user_func($toInt, "5");  // 5 — silent coercion
array_map($toInt, ["5"]);     // [5] — silent coercion
```

A callback passed to `array_map`, `array_filter` or `usort` isn't called by you, but by a built-in PHP function — and that one has no `strict_types`. `call_user_func` is the treacherous one: in a file with a namespace — that is, in any ordinary class — it behaves just like `array_map` unless you write it with a leading `\`. You only get the TypeError with `\call_user_func` (or `use function call_user_func;`), or in a file without a namespace.

## Practical Impact: A Form Never Sends an int

An HTTP form submits **text**. It doesn't matter what `type="number"` you wrote in your HTML — whatever comes into PHP via `$_POST` or `$request->getPost()` is a string. Or an array (for `name="items[]"`), or `null` when the field is missing from the form.

In Nette presenters you have `declare(strict_types=1)` (rightfully so), which means this throws a TypeError:

```php
declare(strict_types=1);

// $request->getPost('quantity') returns "5", not 5
$this->orderService->setQuantity(
    $request->getPost('quantity') // 💥 TypeError
);
```

An explicit cast is the obvious fix. But `(int)` doesn't check anything:

```php
// "abc" → 0, null → 0, no error
$quantity = (int) $request->getPost('quantity');
```

Paradoxically, that makes it looser than coercion itself: `"abc"` passed to an `int` parameter throws a TypeError even without strict_types. If you want invalid input rejected, validate it or use Nette Forms:

```php
// Validation — invalid input (including null or an array) returns false
$quantity = filter_var(
    $request->getPost('quantity'),
    FILTER_VALIDATE_INT,
);
if ($quantity === false) {
    // invalid value
}

// Or Nette Forms — addInteger() validates and converts the value
$form->addInteger('quantity');
// int, or null for an empty field ✅
$quantity = $form->getValues()->quantity;
```

## What strict_types Doesn't Solve: The Falsy "0"

`strict_types` protects you from silent coercion of arguments, but it has no effect at all on conditions, `?:` or `empty()`.

A classic trap: a default value for an empty field.

```php
declare(strict_types=1);

// "If the user leaves it empty, use the default 10 %"
$discount = (int) ($request->getPost('discount') ?: 10);
```

The user enters a 0 % discount and gets 10 %. The string `"0"` is falsy in PHP, just like `""`, `null`, `0` or `[]`, so `?:` skips it. `empty()` has the same problem: it can't tell zero from an empty field.

If zero is a valid value, check for "empty" explicitly:

```php
$raw = $request->getPost('discount');
$discount = ($raw === null || $raw === '') ? 10 : (int) $raw;
```

## Summary

- `strict_types=1` disables silent coercion when passing arguments, returning values and writing to typed properties — mysterious bugs surface as a loud TypeError
- For arguments **the caller's file** decides; for return values it's the file where the function is defined
- It doesn't apply to int → float, operators, comparisons, or callbacks invoked by built-in functions (`array_map` and friends, plus `call_user_func` inside a namespace)
- A form sends a string, an array or null — `(int)` validates nothing, use `filter_var` or Nette Forms
- strict_types replaces a silent bug with a noisy TypeError, but it doesn't replace input validation — logic errors (falsy `"0"`, a negative number where you don't want one) are still on you

The simple rule stands: `declare(strict_types=1)` belongs in every file. But knowing *exactly* how it works will save you a fair bit of confused debugging.
