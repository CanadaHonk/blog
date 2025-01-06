---
title: "Porffor's January 2025 Goals"
description: "A more compliant and stable compiler, while starting foundations for large long-term optimizations."
date: 2025-01-06
draft: false
---

Trying out a new blog series for me setting out [Porffor](https://porffor.dev)'s goals for the month. Hopefully they'll be done by the end! This month has 3 goals:

### 60% Test262
Obviously important for JS compliance, it also helps test stability with its huge (~50k file) test suite. There is definitely a plateau to the curve but I'm reasonably confident I can hit 60% by the end of the month (currently at ~55%).

### Improve general compiler stability by cleaning up and simplifying older code
Only recently when talking to some devs did I realize a hidden advantage of Porffor - it's simplicity. The entire compiler is under 20k lines of code, which is tiny compared to engines like V8 and JSC. This is good not only for maintainability but easy prototyping and future additions. Instead of needing multiple interpreters and JITs, Porffor just compiles to one common target (Wasm) and then more from there instead of needing separate workflows (native, etc).

### Setup larger long-term optimizations
To start, Porffor has a new Wasm optimizer allowing for bytecode-level constant folding and more in future. It's currently off by default due to stability issues but these should be fixed soon. After that, I have more ideas for optimizations not always possible due to spec compliance, but should be able to automagically detect common scenarios it can apply to with static analysis (instead of needing JIT data or observationally breaking spec).

Thanks for reading! As always, you can follow progress and message me on [Twitter](https://x.com/CanadaHonk) or [Bluesky](https://bsky.app/profile/goose.icu) (or [email](mailto:honk@goose.icu)).