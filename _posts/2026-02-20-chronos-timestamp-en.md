---
layout: post
lang: en
page_id: chronos-timestamp
title: "⏰ Why Is Chronos Stealing My Time? Watch Out for Update 3.1.0"
description: "Find out why upgrading the Chronos library to version 3.1.0 can silently shift your database records back by one day, and how to fix this timezone bug elegantly."
date: 2026-02-20 21:00:00 +0100
categories: [Programming, PHP]
tags: [chronos, nette, bug, timezone]
image:
  path: assets/img/posts/chronos-thief.jpg
permalink: /posts/chronos-timestamp/
---

When developing with Nette (or any other PHP framework), we tend to assume that "time is just time." Then a subtle library update arrives and suddenly your database records start showing yesterday's date.

### The Problem: A Midnight That Isn't Midnight

Imagine you have a date stored in your database as `2026-02-04 00:00:00`. But users see `2026-02-03` on the website. Where did that hour go?

The culprit is a combination of **Unix Timestamps** and a change in the **Cake\Chronos 3.1.0** library.

### The Treacherous Timestamp

Code that used to work looked like this:

```php
return Chronos::createFromTimestamp($row->datum->getTimestamp());
```

**What's happening inside:**

1. `$row->datum` is an object (e.g., `Nette\Utils\DateTime`) in the `Europe/Prague` timezone.
2. `getTimestamp()` converts this time to seconds since 1970. **Timestamps are always in UTC.**
3. Midnight in Prague (in winter) means **23:00 of the previous day in UTC**.

### The Change in Chronos 3.1.0 (Preparing for PHP 8.4)

Previously, Chronos would try to "guess" the timezone from the server settings when creating an object from a timestamp. Since version 3.1.0, a fundamental change was introduced:

> If no timezone is explicitly passed, `createFromTimestamp()` now strictly returns an object in **UTC (+00:00)**.

This means you get an object with time `23:00:00 UTC`. As soon as you call `->format('Y-m-d')` on it, it outputs yesterday's date.

### How to Fix It?

#### 1. The Elegant Way (Recommended)

If you already have a `DateTime` object (which Nette Database returns automatically), don't use a timestamp as an intermediary. Use the `instance()` method instead:

```php
// Chronos extracts both the time and the correct timezone (Prague) from the original object
return Chronos::instance($row->datum);
```

#### 2. The Explicit Way

If you truly only have a raw timestamp (a number), you must tell Chronos in which timezone to interpret it:

```php
return Chronos::createFromTimestamp($ts, 'Europe/Prague');
```

### Lessons Learned

* **Timestamps are treacherous:** When converting between local time and a Unix timestamp, you always risk losing timezone information.
* **Read changelogs:** Even a minor update (3.0 → 3.1) can contain a behavior change that silently breaks existing logic.
* **PHP 8.4:** This change in Chronos isn't random — it prepares the ecosystem for standardization coming directly to the PHP core.
