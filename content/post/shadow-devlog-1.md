---
title: "Shadow Devlog #1: 1 week in"
description: "What's new in Shadow since the intro post (Oct 27-30)"
date: 2023-10-30
draft: false
---

<div class="toc">

### Table of Contents

- [JavaScript](#javascript)
  - [Slaying the Kraken 🐙](#slaying-the-kraken-)
- [Comparisions](#comparisons)
  - [SerenityOS: 2nd birthday page](#serenityos-2nd-birthday-page)
  - [Kraken: Start page](#kraken-start-page)
- [Layout](#layout)
  - [Text Wrapping](#text-wrapping)
  - [CSS Selectors Rewrite](#css-selectors-rewrite)
  - [External Stylesheets](#external-stylesheets)
  - [`margin: auto`](#margin-auto)
  - [Beginning of `<iframe>`](#beginning-of-iframe)
  - [`color-scheme`](#color-scheme)
  - [Rewritten `<img>`](#rewritten-img)
  - [`position: absolute`](#position-absolute)
  - [`text-align`](#text-align)
  - [`<br>`](#br)
  - [Tweaks](#tweaks)
- [Misc](#misc)
  - [HTML Parser Improvements](#html-parser-improvements)
  - [Loading Non-HTML](#loading-non-html)

</div>

Welcome to the first Shadow devlog! If you aren't aware, [Shadow is my novel browser engine made almost entirely in JS](/introducing-shadow/). I'm aiming to do these once a week to cover the last week's progress, although here I'll cover since the prior intro post (Oct 27-30). Shadow is 1 week old as of this post! 🎉

### Notices

- Minor rebrand from `<shadow>` to Shadow, mostly just to make things simpler.
- The [Introducing Shadow](/introducing-shadow/) post hit the top of Hacker News, which was pretty neat (and luckily mostly nice).

<br>

## JavaScript

JS support got completely rewritten *(twice)* and is now usable in websites! We now run the JS backend (host/`eval`, or [SpiderMonkey](https://spidermonkey.dev)/[Kiesel](https://kiesel.dev) via Wasm) in a web worker via a mix of `postMessage` and `SharedArrayBuffer`/`Atomics`.

`<script>` and `<body onload=...>` are also newly supported.

Bonus thanks to [Linus](https://linus.dev) for their work on [Kiesel](https://kiesel.dev) (supported JS backend by Shadow) to help it get working. They [wrote a post on the work](https://linus.dev/posts/kiesel-devlog-5/), check it out!

### Slaying the Kraken 🐙

Most notably, we can now run the older JS benchmark, [Kraken](https://mozilla.github.io/krakenbenchmark.mozilla.org/), even navigating and rendering results as expected! (Don't take the results seriously, it is just an old synthetic JS benchmark).

Here's a **sped-up** GIF of it running:

![GIF showing Kraken running in Shadow](/img/kraken.gif)

<br>

## Comparisons

(left/top: now, right/bottom: last post)

### SerenityOS: 2nd birthday page

With text wrapping, `position: absolute`, and `text-align` support - it looks very good now!

<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/52d567a9-ce7d-4cfd-91aa-64e47926c518" width=500>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/2cd18e82-c0d0-4591-bf72-391d4810fd4e" width=500>
</p>

### Kraken: Start page

It was completely broken before, now is essentially perfect!

<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/05763088-77f4-41ed-add9-4c2e93c8b097" width=500>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/9795c13c-bf4a-41ae-b831-84508795603b" width=500>
</p>

<br>

## Layout

### Text Wrapping

This was the most noticable update since the post. Shadow now wraps text! It was actually somewhat easier than I expected, although I had my fair share of debugging random position jank. It should be mostly stable now though. It's even responsive/dynamic!

![GIF showing text wrapping on a page changing depending on the width of the page](/img/wrapping.gif)

### CSS Selectors Rewrite

I rewrote both parsing and testing CSS selectors to now support combinators (eg `#a .b`, `div > span`).

![Screenshot of demo page](https://github.com/CanadaHonk/shadow/assets/19228318/8afdb27e-eb14-4ad5-8442-be924b83b5d1)

```html
<div id="a">a
  <div id="b">b
    <div id="c">c
      <div id="d">d</div>
    </div>
  </div>

  <div id="e">e</div>
</div>

<style>
  #a {
    color: purple;
  }

  #a div {
    color: red;
  }

  #a > div {
    color: blue;
  }

  #a > div > div div {
    color: green;
  }
</style>
```

### External Stylesheets

Shadow now (tries to) load external stylesheets if specified with `<link rel="stylesheet" href="...">`. This is good, but also sometimes unfortunate as we try to load CSS we don't support and crash the tab (todo: gracefully handle CSS parser errors).

### `margin: auto`

I added support for `margin: auto` so elements are now centered in the parent as expected.

![Screenshot of the Kraken start page showing it centered with an inspect overlay](https://github.com/CanadaHonk/shadow/assets/19228318/fe113507-435d-4919-b1fb-ba3cc5dab34d)

### Beginning of `<iframe>`

We also now have the beginning of an `<iframe>` implementation! We currently do not respect `src` (!) rather just treat it as empty and as a separate container.

### `color-scheme`

Shadow now supports the `color-scheme` CSS property, and `<meta>` name. This was added mostly to avoid old/light pages to use dark default colors if Shadow was in dark mode. We now respect the `color-scheme` of the page, and the user's preferred scheme, but only if supported.

<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/e0177a83-928c-4dd0-9602-78721ddbd1e5" width=500>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/9901b9da-56ae-4ff5-9705-42c2f8405a89" width=500>
</p>

### Rewritten `<img>`

Shadow now properly respects the image's `Content-Type`, previously many images were broken as it was always expecting PNGs. Also, if an image is broken it no longer crashes the renderer. Example with an SVG:

![Screenshot of the Shadow welcome page showing an SVG image](https://github.com/CanadaHonk/shadow/assets/19228318/89c78a02-d835-43a6-9694-78c0babb7b7b)

### `position: absolute`

I added basic support for `position: absolute`, it is quite limited for now but does its job for now ;)

### `text-align`

With quite a hack, `text-align` is now implemented too (`center` and `right`), still a bit unstable.

![Screenshot of a page showing text and an image centered in a div](https://github.com/CanadaHonk/shadow/assets/19228318/d8b93e10-c734-4729-acd1-dfb4e4c1961d)

### `<br>`

Alongside text wrapping, Shadow now supports breaking lines with `<br>`.

![Screenshot showing terminal-like text with broken lines](https://github.com/CanadaHonk/shadow/assets/19228318/3b9a6bcb-6a58-49aa-8dad-8bd677c83dc2)

### Tweaks
- Default unit to `px` (previously errored)
- Unescape more encodings (eg `&#00A0`)
- Tweaked when Shadow displays the page
- Improved collapsing vertical margin

<br>

## Misc

### HTML Parser Improvements

- Now ignores HTML comments
- Now excludes content of `<style>` and `<script>` elements
- No longer errors when trying to insert `<html>` if no child elements are generated

### Loading Non-HTML

I started working on loading non-HTML files, like CSS or JS source files by just generating a small HTML wrapper of the text:

![Screenshot of Shadow rendering it's own user agent CSS file](https://github.com/CanadaHonk/shadow/assets/19228318/bf6ad7d0-e14a-4cd9-88b1-f7a2591497ff)

<br>

Thanks for reading! Hope you found it interesting. Tune in next week (🤞) for this week's progress, or [look at my Twitter](https://twitter.com/CanadaHonk) for "live" updates.