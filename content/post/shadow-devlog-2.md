---
title: "Shadow Devlog #2"
date: 2023-11-06
draft: false
tangent: true
---

<div class="toc">

### Table of Contents

- [Comparisons](#comparisons)
  - [Ladybird](#ladybird)
  - [SunSpider Results](#sunspider-results)
  - [WHATWG](#whatwg)
- [Performance](#performance)
  - [New Debug Overlay](#new-debug-overlay)
  - [Page Load Profiler](#page-load-profiler)
  - [~3x Faster Page Load](#3x-faster-page-load)
  - [~2x Faster Rendering](#2x-faster-rendering)
- [Layout](#layout)
  - [Pseudo-classes](#pseudo-classes)
  - [`var()`](#var)
  - [CSS Parser](#css-parser)
  - [~8x Faster Dead Text Node Removal](#8x-faster-dead-text-node-removal)
  - [Many Fixes](#many-fixes)
- [JavaScript](#javascript)
  - [~5x Faster Worker Execution Loop](#5x-faster-worker-execution-loop)
  - [New APIs](#new-apis)
- [HTML Parser](#html-parser)
  - [Attributes Without Value](#attributes-without-value)
  - [Autofix Mismatching Closing Tags](#autofix-mismatching-closing-tags)

</div>

Welcome to the second Shadow devlog! If you don't know it, [Shadow is my novel browser engine made almost entirely in JS](/introducing-shadow/). Shadow is 2 weeks old as of this post! 🎂

Shadow was (accidentally) in the news again! This time in [JavaScript Weekly #661](https://javascriptweekly.com/issues/661).

<br>

## Comparisons

(left/top: now, right/bottom: last post)

### Ladybird

It was completely busted before, but now is quite good! The header looks broken as we do not have flex support yet.

<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/c2078318-4afb-490f-a00b-b7b067b213c4" width=500>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/398dba69-ba25-4117-b885-a2b9792c6a4b" width=500>
</p>

### SunSpider Results

The results did not display before as it used unsupported DOM APIs, but we now implement them so it shows fine now. Also looks a bit better due to some layout fixes.

<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/b00f8856-21ba-4cd7-9b85-f40e562cbd91" width=500>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/a0f5b048-6286-4c70-afaf-c20e91cde3f0" width=500>
</p>

### WHATWG

Before it was completely broken, but now it no longer crashes! It doesn't look great but progress is progress :)

<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/34e457a4-3e0f-4e0d-8c98-df95fcf5ad89" width=500>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/178d7c40-f5fe-4ce3-baa9-85c38b6bbc77" width=500>
</p>

<br>

## Performance

I did a lot of work on performance and debug tools to help investigate it. I mostly cared about load time performance but did a bit of work on rendering too.

### New Debug Overlay

Shadow has a new debug overlay, showing frame time and delta time (accessed via Shift+Z).

![Screenshot of new debug overlay](https://github.com/CanadaHonk/shadow/assets/19228318/3fc4ca01-ee6d-4e15-babd-e49af0e675a4)

### Page Load Profiler

Additionally, we now profile page loads to see what takes up the time.

![Screenshot of page load profiler](https://github.com/CanadaHonk/shadow/assets/19228318/8ae99c77-7c22-48aa-a663-494f1b2731f3)

### ~3x Faster Page Load

Compared to before, page loads are now ~3x faster! This is mostly thanks to [rewriting script execution](#5x-faster-worker-execution-loop) and [removing dead text nodes](#8x-faster-dead-text-node-removal).

### ~2x Faster Rendering

Rendering the page is also ~2x faster than before, thanks to adding offscreen culling (not rendering elements outside of the viewport).

<br>

## Layout

### Pseudo-classes

Shadow is now beginning to implement pseudo-classes. We currently implement `:link` (+ `:any-link`) and `:root`.

### `var()`

Shadow now supports the `var()` CSS function, including a default value (`var(foo, 2px)`).

### CSS Parser

The CSS parser now also supports psuedo-classes, and can now parse at-rules so no longer crashes when encountering them (although they are just ignored in layout for now). It also now bails instead of crashing the entire tab when it errors.

### ~8x Faster Dead Text Node Removal

Removing dead text nodes is a pass layout does after assembling the layout tree to simplify it for us later. It removes unneeded text nodes (empty, or pure whitespace which can be appended to others).

I rewrote it to be ~8x faster which noticably helps page load times with big pages.

### Many Fixes

- Fixed some CSS selector matching bugs
- Fixed % values for position absolute
- Rewrote computing element's height

<br>

## JavaScript

### ~5x Faster Worker Execution Loop

The worker execution loop (how the worker loops into getting JS to eval) is now async, so it no longer sits there constantly using CPU when not executing JS. This also makes execution ~5x faster due to not having to wait a loop cycle.

### New APIs

`appendChild`, `createElement`, and `createTextNode` are now implemented, also some of `performance`, plus `queueMicrotask`.

<br>

## HTML Parser

### Attributes Without Value

The HTML parser can now parse attributes without a value, eg `<div foo>`.

### Autofix Mismatching Closing Tags

The HTML parser now also tries to automatically fix mismatching closing tags. For example:

```html
<main>
  <ul>
  <li>item
</main>
<article>
</ul>
</li>
</article>
```

Gets transformed into:

```html
<main>
<ul>
  <li>item</li>
</ul>
</main>
<article>
</article>
```

<br>

Thanks for reading! Hope you found it interesting. Tune in next week (🤞) for this week's progress, or [watch my Twitter](https://twitter.com/CanadaHonk) for "live" updates.
