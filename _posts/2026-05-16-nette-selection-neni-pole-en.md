---
layout: post
lang: en
page_id: nette-selection-neni-pole
title: "🔥 fetchAll() in DataGrid? You Just Killed the Server — Why Selection Isn't an Array"
description: "Pulling data with fetchAll() too early can reliably take down your server. Learn how to filter smartly in Nette DataGrid using SQL-driven approaches."
date: 2026-05-16 20:15:00 +0200
categories: [Programming, PHP]
tags: [php, nette, database, optimization, datagrid, contributte]
image:
  path: assets/img/posts/2026-05-16-nette-selection-database.png
permalink: /posts/nette-selection-neni-pole/
---

Has this ever happened to you? You're building a datagrid, filters for an admin panel, or a data export. You create a query through `Nette\Database`, want to pass it to the grid, and with good intentions you write this:

```php
// ❌ WRONG: You're handing the grid a fixed array of fully loaded rows
$orders = $this->database->table('order')->fetchAll();
$grid->setDataSource($orders);
```

Everything works great... until the web server runs out of memory in production, the CPU burns, and users are greeted with a **500 Internal Server Error**.

Let's look at why this happens and why understanding the following is the key to success — `Nette\Database\Table\Selection` is not an array of data, it's a **recipe**:

```php
// ✅ CORRECT: You hand the grid only a "recipe" — it fetches the data itself at the end
$orders = $this->database->table('order');
$grid->setDataSource($orders);
```

## Recipe vs. Ready Meal

When you call `$orders = $this->database->table('order');` in Nette, the variable `$orders` does **not** contain thousands of rows from the database. It doesn't contain a single one.

`Selection` is basically just an "order ticket" or a recipe for the database. You're saying: *"When you need something from me, know that we'll be going to the orders table."*

When you add a condition:
```php
$orders->where('status', 'paid');
```
Still nothing has been fetched from the database. You've just added a note to the order ticket: *"...and I only want the paid ones."*

Notice that when you pass the selection to the grid via `$grid->setDataSource($orders)`, you're still only passing the **recipe**. During its lifecycle, the grid adds its own ingredients — like whatever the user is currently searching for in the filters, or which page they clicked (it adds its own `LIMIT` and `OFFSET`).

The actual SQL query is fired at the very end, when something tries to pull the data out of the `Selection` for the first time to render the table.

## Killing the Server in Real Time (Anti-pattern)

Imagine you're implementing a search in an admin panel for orders. You want to filter by customer name. But you have different types of orders (e.g., for companies and individual customers) with names in different tables, so you decide to handle it "conveniently" in PHP:

```php
$grid->addFilterText('customerName', 'Customer Name')
    ->setCondition(function (Selection $selection, string $value) {
        // This forces Nette to actually load ALL orders into PHP memory
        $allOrders = $selection->fetchAll();

        $filteredIds = [];
        foreach ($allOrders as $order) {
            // Load a complex entity and filter in PHP
            $customerName = $order->ref('customer')->name;
            if (str_contains(mb_strtolower($customerName), mb_strtolower($value))) {
                $filteredIds[] = $order->id;
            }
        }

        // Pass the filtered IDs back to the selection
        $selection->where('id', $filteredIds);
    });
```

### Why Is This the Road to Hell?
If you have 500 orders in the shop, the app will be lightning fast. But with 50,000, here's what happens:
1. **PHP Memory Limit:** You trigger a `SELECT * FROM order` query that dumps a massive chunk of data into PHP's RAM. PHP runs out of memory and the script immediately crashes with `Fatal error: Allowed memory size exhausted`.
2. **Wasted CPU cycles:** The server's CPU is frantically comparing thousands of strings in a `foreach` loop, which takes seconds. Users stare at a spinning loader.
3. **Wasted work:** You fetched all that data just to extract a handful of IDs and then query the database again for essentially the same thing.

## The Right Way: Let the Database Do the Work

Databases (MySQL, PostgreSQL, whatever) are incredibly optimized engines. They're built to scan millions of rows, join tables, and filter data at lightning speed. Your job as a programmer is to hand them the right **recipe**.

Instead of pulling data into PHP and sifting through it there, modify the SQL query inside the `Selection` directly:

```php
$grid->addFilterText('customerName', 'Customer Name')
    ->setCondition(function (Selection $selection, string $value) {
        // Just modify the "recipe" (the SQL query) using EXISTS and parameters
        $selection->where("
            EXISTS (
                SELECT 1 FROM `customer` `c`
                WHERE `c`.`id` = `order`.`customer_id`
                AND `c`.`name` LIKE ?
            )
        ", "%$value%");
    });
```

### What Happened Now?
**Not a single extra row** was loaded into PHP memory. Nette didn't pass data to the grid — it passed a modified *recipe*. The datagrid appends `LIMIT 20 OFFSET 0` (for pagination) to this recipe at the end and only then fires the query to the database.

The database scans its indexes, instantly picks exactly those 20 rows the user wants to see on the first page, and only those 20 rows travel into PHP. The result? **Processing takes 0.02 seconds and memory usage is minimal.**

## Summary for Your Next Project

Whenever you work with `Selection` in Nette, stick to the rule: **PHP only displays data, the database filters and sorts it.**

* If you're filtering data in a component (like a grid), **never** use `fetchAll()` or loop over the entire table.
* Leverage the power of SQL (`WHERE`, `JOIN`, `EXISTS`).
* Pass values as parameters (`?`) — Nette handles security (SQL Injection) and proper string escaping for you.

Your servers will thank you, and your users will get an instantly loaded page instead of a 500 error.
