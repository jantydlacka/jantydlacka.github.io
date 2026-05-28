---
layout: post
lang: en
page_id: https-redirect-browser
title: "🔓 How to Remove a Permanent HTTPS Redirect in Your Browser"
description: "A quick guide to fixing an unwanted permanent HTTPS redirect on localhost in Brave and Chrome."
date: 2026-03-25 14:00:00 +0100
categories: [Programming, Web]
tags: [localhost, https, devtools, hsts, brave, chrome]
image:
  path: assets/img/posts/2026-03-25-https-redirect-browser.png
permalink: /posts/https-redirect-browser/
---

It happens to the best of us. You're tweaking `.htaccess` on localhost, you set up an "experimental" redirect to HTTPS using a **301 (Permanent)** status code — and then you discover that even after deleting the rule, the browser simply won't go back to HTTP.

Here's a quick guide to get yourself unstuck.

### 1. First Aid: DevTools + "Disable Cache"

This is the fastest path if you need to see changes immediately without doing a deep browser cleanup.

1. Open **Developer Tools** (`F12`).
2. Go to the **Network** tab.
3. Check the **Disable cache** checkbox.

As long as the DevTools panel is open, the browser will ignore cached redirects and load the page fresh from the server.

### 2. Hard Reload

If you want this to keep working after closing the Developer Tools, you need to explicitly evict the cache for that page:

1. Keep **DevTools** open (`F12`).
2. Right-click the page reload icon (the circular arrow in the address bar).
3. Select **Empty Cache and Hard Reload**.

### 3. Deep Clean: Deleting the HSTS Domain Entry

Sometimes the browser decides that localhost belongs to its list of trusted HTTPS-only sites (HSTS) and refuses to talk about anything but HTTPS.

1. Paste this into your Brave/Chrome address bar: `brave://net-internals/#hsts` (or `chrome://net-internals/#hsts`).
2. Scroll down to the **Delete domain security policies** section.
3. Type `localhost` in the **Domain** field and confirm with the **Delete** button.

### Why Did This Happen? (A Lesson for Next Time)

The root cause was using a **301** status code. It tells the browser: "This site has permanently moved here — never come back asking for HTTP again." The browser writes this to its on-disk database and stops asking the server the next time.

> **The developer's golden rule:**
>
> When testing on localhost, always use a **302 (Found / Temporary)** redirect in `.htaccess`. That tells browsers to ask again next time, keeping you in control.
