---
title: "1 year of Porffor"
description: "My JS engine is a year old! Let's recap what has happened and what the future is."
date: 2024-06-25
draft: false
---

[Porffor](https://porffor.dev) is a year old as of today! 🎂 Porffor is my hobby JS engine which is quite unique; it compiles JS to WebAssembly ahead-of-time! This is unlike major JS engines which either interpret or just-in-time compile JS. I'm using this different approach to research the potential of it and what it could allow as a novel JS engine/compiler, plus [just for fun](https://justforfunnoreally.dev).

<img alt="Screenshot of terminal showing Porffor running and compiling a hello world" src="https://github.com/CanadaHonk/porffor/assets/19228318/de8ad753-8ce3-4dcd-838e-f4d49452f8f8" width="70%">


## Recap

Let's recap each major update to Porffor since around January:
- **Porffor now has objects!** Yep, they were not supported before due to their polymorphic nature being complex to handle internally, but since ~last week it now has them!
- **Indirect calls.** Previously, Porffor did not support "indirect calls" - calls which the compiler did not know what was being called at compile-time. It is now supported, allowing callback arguments and more!


## Test262

[Test262](https://github.com/tc39/test262) is the official ECMAScript conformance test suite most engines use for spec compliance testing. On January 31st, Porffor was passing 11.13% of all of Test262. **Today, it is passing 21.65%**! That is almost doubling in just these last few months, and the rate is increasingly exponentially!

<img alt="Graph of Porffor Test262 over time" src="https://github.com/CanadaHonk/porffor/assets/19228318/5320da7d-e945-4d16-857b-499f3a6c1180" width="100%">


## Native compilation

Since we are compiling JS to Wasm AOT, why can't we compile to native binaries too? We can! I wrote my own custom Wasm to C compiler specialized for Porffor, so it can now compile JS to native binaries which just run on your system. Not only are the binaries tiny (<20KB) but they are fast (>2x fast as Node in some tests) too, also consuming much less memory (1-2MB).

<img alt="Screenshot of a terminal showing Porffor compiling and using a cat program" src="https://github.com/CanadaHonk/porffor/assets/19228318/e0581394-ba52-415b-9f56-7649558ca306" width="70%">

<img alt="Screenshot of a terminal showing Porffor compiling and using a Brainf interpreter program" src="https://github.com/CanadaHonk/porffor/assets/19228318/c4c19816-1f5c-4e9f-9c69-00e9be5597d3" width="70%">


## Future

The main current goal is just to make more JS work with it so more scripts can run, also improving our spec compliance. Also ensuring we can continue to compile to tiny+fast native binaries along the way. Maybe one day we can compile some JS CLIs to native? We'll see!

Stay tuned on development via [Twitter](https://x.com/canadahonk) and/or [Mastodon](https://donotsta.re/honk). Interested in contributing? Message! Thanks for reading!