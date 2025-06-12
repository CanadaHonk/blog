---
title: "JS engine code size"
date: 2025-05-21
draft: false
tangent: true
---

Quick tiny blog post for the size of many JS engines because I couldn't find it myself and took the rabbit hole of measuring them myself. Some asterisks, this was a quick measurement and I'm not saying they are exact or correct; but I think it gives a good ballpark. These sizes are just the core source of the engine, doesn't include some parts intentionally (eg shared libraries or dependencies):

| Engine | Popular uses | Ballpark lines of code (code lines only, not just comments or blank space) |
| ------ | ------------ | ------------- |
| V8 | Chromium, Node, Deno | ~1.6 million |
| JavaScriptCore | Safari, Bun | ~800 thousand |
| ChakraCore | EdgeHTML (rip) | ~700 thousand |
| Hermes | React Native | ~160 thousand |
| LibJS | [Ladybird](https://ladybird.org) | ~150 thousand |
| QuickJS | Widely used as a small(er) interpreter | ~80 thousand |
| [Porffor](https://porffor.dev) | Yours truly, more soon(tm)! | ~20 thousand |


I was looking at these as when talking to a few engineers one part of Porffor that stuck out to them was how tiny it is compared to other JS engines. While yes it will definitely grow over time, I'm pretty confident it will never reach the sizes of V8 or JSC. This gives Porffor the great advantage of both easier maintenance and prototyping!
