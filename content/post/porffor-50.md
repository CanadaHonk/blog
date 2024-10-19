---
title: "Porffor passes 50% of Test262"
description: "Test262 is the official ECMAScript conformance test suite; Porffor now passes 50% of it!"
date: 2024-10-19
draft: false
---

50%! 🥳 This is a huge milestone and I like to think shows the promising potential of [Porffor](https://porffor.dev). Truly AOT compiling of JS to Wasm/native has long been attempted and dismissed as infeasible, but this for the first time proves it can be possible in a new JS engine.

### What does that mean?

Test262 is the official ECMAScript conformance test suite, it has almost 50k tests for every feature in the language, down to very niche details. Porffor now passes half of all of these! For a comparison: V8, JavaScriptCore, SpiderMonkey all pass around 88%; Hermes passes around 55%.

### What is next?

A shifting of priorities. Porffor development will now not purely focus on conformance as it has been for the past ~year. Instead for now, it will be more like 50/50 between stability and performance with a temporary feature freeze. Stability is Porffor's largest problem, as expected for an early unstable project. I would say this is the current main blocker of using Porffor in production (no longer conformance!).

### A review of current Test262 results

You may have seen Porffor's Test262 output before:

![Screenshot of Porffor's Test262 output](https://github.com/user-attachments/assets/e0258c88-0b7b-47a3-8ff3-7c49d76e9a58)

```
test262: 50.15% | 🧪 48381 | 🤠 24261 | ❌ 6951 | 💀 15573 | 🏗️ 152 | 💥 331 | ⏰ 135 | 📝 978
```

But what do those emoji and numbers mean? Let's break it down:

- `🧪 48381`: The total number of tests.
- `🤠 24261`: Passes!
- `❌ 6951`: Failed gracefully (eg an assert failed).
- `💀 15573`: Runtime error (eg a TypeError was thrown).
- `🏗️ 152`: Malformed Wasm output from the compiler.
- `💥 331`: Compiler error.
- `⏰ 135`: Timed out (took >10s).
- `📝 978`: Hit a compiler todo error (eg bigint literal, `import`).

The good part is the very bad things (malformed Wasm and compiler errors) are very infrequent, now under 1%, showing the improvements in stability. The number of timeouts is not great, but there are some huge tests and the timeout limit is relatively short at 10s so I am not that concerned.

### Progress over time

![Graph of Porffor Test262 stats over time since inception](https://github.com/user-attachments/assets/fea2692d-2906-4042-812e-ae6633ff275c)

Thanks for reading! You can [follow progress](https://x.com/CanadaHonk), [message](https://x.com/CanadaHonk) or [email](mailto:honk@goose.icu) me in these places.