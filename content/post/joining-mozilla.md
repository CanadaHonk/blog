---
title: "Joining Mozilla"
description: "On October 1st, I join Mozilla to work on Firefox full-time!"
date: 2023-09-30
draft: false
---

Hey! Big news first: **I join Mozilla on October 1st to work on Firefox full-time**! Specifically a software engineer on the [DOM Core team](https://wiki.mozilla.org/DOM/Core). I am extremely excited to work on the web platform full-time. This post covers the basics of who and how. If you have any questions feel free to [DM me](https://twitter.com/CanadaHonk). (Good luck job hunters!)

### Who

I go by `CanadaHonk` (I prefer not to share my full name publicly/obviously online). Summary of yours truly:
- I live in the UK (but have no particular allegiance)
- My programming knowledge is entirely self-taught in my free time. I have not been to (UK) university/etc (and do not plan to for now).
- My free time is spent largely on programming open source projects (my own and others, large and small). Mostly Firefox (more below), [my JS compiler](https://porffor.goose.icu), and more recently [Node.js](https://github.com/nodejs/node/pulls?q=is%3Apr+author%3ACanadaHonk).

### How

Since about February this year, I began contributing to Gecko (Firefox's web engine). I started by fixing some small [CDP](https://chromedevtools.github.io/devtools-protocol/) (Chrome's remote debugging protocol) bugs I had encountered while working on a personal project. I then got hooked and did more complex things like adding some new commands/features which I found handy.

Next I began branching out: doing a bit of work on Necko (Gecko's network stack); and layout/Stylo ([Servo](https://servo.org)'s style system in Gecko). I slowly started doing more and more layout work until I am mostly just working on that.

Finally, I went from only smaller/simpler fixes to implementing new web platform features. Here's a list of notable things I've added to Firefox this year in my free time:
- `@media (scripting)` (113)
- CSS `NaN`/`infinity` fixes + ship (114)
- `@import supports()` (115)
- `URL.canParse()` (115)
- `<search>` (118)
- `attr()` fallback (119)
- `@media (prefers-reduced-transparency)` (behind pref)
- `@media (inverted-colors)` (behind pref)
- NVIDIA Video Super Resolution (behind pref)

If you like stats; I was the ~5th highest individual contributor this year by commits [(source)](https://bkardell.com/blog/2023-Mid-Season-Power-Rankings.html), as of writing, I have:
- Submitted 93 patches
- Filed 80 bugs
- Filed 9 intents (2 prototype, 3 ship, 4 prototype+ship)

A few months ago, a Mozillian suggested I try to get a job there, then I started talking with some people about a potential contract... and now here we are! Still feels quite surreal. Thank you to many Mozillians for supporting me, reviewing patches, and just generally being great people! :)