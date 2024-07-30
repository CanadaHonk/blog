---
title: "A new chapter for Porffor"
description: "From next week, I will be independently building Porffor full-time!"
date: 2024-07-30
draft: false
---

[Porffor](https://porffor.dev) is my hobby ahead-of-time JS engine — but no longer! This week I have resigned [from Mozilla](/2024/) after a wonderful 10 months to work on Porffor full-time, supported by [Chris Wanstrath](https://x.com/defunkt) for one of his unannounced projects.

### What is Porffor?

If you don't know, Porffor compiles JavaScript (or TypeScript) code to WebAssembly ahead-of-time, rather than interpreting or just-in-time compiling like existing engines do. Compiling JS AOT allows some exciting possibilites: shipping JS as Wasm instead of needing to ship a full JS engine; better safety via Wasm sandboxing; massive performance improvements compared to interpreting; potential native compilation; and more!

### What changes?

Other than my full attention being on Porffor, development will remain the same: independent and open source! The main focus for the moment remains to be ECMAScript conformance, measured via continuous [Test262](https://github.com/tc39/test262) (the official ECMAScript conformance test suite) runs. Porffor is currently passing almost 35% of the test suite!

If you have any questions or just want to chat about Porffor, please feel free to [message](https://x.com/CanadaHonk) or [email](mailto:honk@goose.icu).