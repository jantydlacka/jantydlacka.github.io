---
layout: post
lang: en
page_id: mdbrowse
title: "📄 How to Read Markdown on Linux in Your Browser"
description: "Double-clicking a .md file opens an editor, and a browser shows only raw text. mdbrowse turns it into a GitHub-style page — read-only, no internet needed."
date: 2026-10-20 08:00:00 +0200
categories: [Programming, Linux]
tags: [markdown, linux, bash, open-source]
image:
  path: assets/img/posts/2026-10-20-mdbrowse.png
permalink: /posts/mdbrowse/
---

Markdown is everywhere now. A README in every repo, documentation, Obsidian notes, whatever your AI tools spit out. On GitHub it looks great — headings, tables, code blocks, images. But double-click the same file on your disk and you get an editor. Or a wall of `##` and `**asterisks**`. You wanted to *read* it, not edit it.

What's missing is the most ordinary thing imaginable: a viewer. You have one for images. You have one for PDFs. For Markdown, you don't. Your web browser would be the perfect candidate, but it can't render Markdown — it shows you the raw text. Your IDE renders it fine, but launching a full IDE to read one file is like using a sledgehammer to crack a nut.

So I wrote that viewer. It's called [mdbrowse](https://github.com/jantydlacka/mdbrowse){:target="_blank" rel="noopener"}.

## What it does

mdbrowse registers itself as the default application for `.md` files. From then on, double-clicking any Markdown file opens your browser with a rendered page — looking like GitHub, light and dark mode included. Nothing more. You can't type into it, which is the point: you can't accidentally clobber the file either.

![A raw .md file rendered by mdbrowse into a GitHub-styled page](assets/img/posts/2026-10-20-mdbrowse.gif)

No magic inside: your browser can't render Markdown, but it renders HTML just fine. So mdbrowse wraps your file into one self-contained HTML page (carrying its own parser, styles and sanitizer), writes it to a cache directory and opens it. Everything runs locally — no server, no API, no telemetry, works entirely offline.

## Install

```sh
git clone https://github.com/jantydlacka/mdbrowse.git
cd mdbrowse
./install.sh
```

The whole tool is a single bash script of roughly 500 lines, about a quarter of them comments. No Python, no npm, no binary you have to take on faith — you can read all of it. `./uninstall.sh` puts everything back, including your original `.md` handler.

## How it differs from similar tools

There are other "Markdown to browser" tools — mdview, mdopen, grip ([the README](https://github.com/jantydlacka/mdbrowse#alternatives){:target="_blank" rel="noopener"} has an honest comparison; some may suit you better). Two things set mdbrowse apart:

**You can click through a whole tree of notes.** Got an index linking ten other files? The links work — mdbrowse pre-renders the targets recursively and rewrites the links to point at each other, heading anchors included. Tools that, like mdbrowse, just generate a page (mdview, for one) open one file and leave you there. The ones that let you click through run a local server, like grip or mdopen — mdbrowse does it without one.

**It works where your browser can't reach.** A browser from Snap — on Ubuntu that includes the default Firefox — runs in a sandbox and **isn't allowed into hidden folders** directly in your home directory. Notes under `~/.config/` or `~/.notes/` simply won't open. mdbrowse reads the file itself and writes the rendered page somewhere the sandbox *can* reach. You don't need to know any of this — mdbrowse detects it on every run.

## Security

And one thing you wouldn't expect from a notes viewer until it bites you: Markdown is allowed to contain raw HTML. A single `<img src="logo.png" onerror="…">` line in a downloaded file is enough to run someone else's JavaScript after a double-click — with network access, so it can, for example, send the page's content to someone else's server. And all you did was "open the file".

That's why mdbrowse runs all HTML through a sanitizer (DOMPurify) before it reaches the page. Of the compared tools that render locally, it's the only one that does — verified in their source code, not their READMEs. grip only gets clean HTML because it sends your file to GitHub to render. The full breakdown, including a sample hostile document, is [in the Security section](https://github.com/jantydlacka/mdbrowse#security){:target="_blank" rel="noopener"}.

## What it doesn't do

You read in it, you don't write. No syntax highlighting in code blocks, no Mermaid diagrams, no math, Linux only, and the rendered page needs JavaScript enabled. If you want hot reload while writing, take mdopen (with `--reload`); if you want Mermaid, mdview.

## Try it

The repo is at [github.com/jantydlacka/mdbrowse](https://github.com/jantydlacka/mdbrowse){:target="_blank" rel="noopener"}, MIT licensed. Built and tested on Ubuntu with GNOME; it relies only on XDG conventions, so it should run on any common desktop. If you try it on Fedora, Arch or KDE, I'd love to hear how it went in the [GitHub issues](https://github.com/jantydlacka/mdbrowse/issues){:target="_blank" rel="noopener"} — especially if something breaks.
