---
title: "Porffor: Objects v2"
description: "Porffor's rewritten object implementation is now much faster and produces 2-5x smaller Wasm binaries when using objects!"
date: 2025-02-05
draft: false
---

[Porffor](https://porffor.dev)'s object implementation was never great; it was a map without even hashing to get something simple done well. Luckily, Porffor's new object implementation is complete! Not only is it faster, it also makes much smaller Wasm than before and is more conformant!

- **Objects finally have hashed keys** via my own [FNV-1a](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function#FNV-1a_hash) implementation. It is a pretty basic implementation but I have some ideas for future improvements (better string type compatibility, SIMD variant maybe).

- **Objects have their prototype cached inline**, previously objects had to do a lookup to access their own prototype but now it is just 1 read!

- **Rewritten built-in/hidden class prototype lookup**, this is quite under-the-hood and complicated. If you do eg `[].forEach(...)`, Porffor does not actually compile that to the equivalent of `Object.getPrototypeOf([]).forEach.call([], ...)` as you would likely expect. Instead of looking up the prototype and having to do unnecessary object things, the compiler knows to just do `Array.prototype.forEach.call([], ...)` and to call it directly with the function not via `Array.prototype`. This also saves a lot of Wasm size as we don't have to include all of `Array.prototype` for just one prototype function call. However, we have to actually do that if someone were to do `Object.getPrototypeOf([])`. Previously, Porffor had a design flaw: if you deal with objects we practically always have to include all of `Object.prototype`, including `Object.prototype.isPrototypeOf` which would include almost every prototype regardless if actually used or needed as it could not know from just that function's context. The rewrite fixes this design flaw making most JS files produce 2-5x smaller Wasm binaries!

`richards.js` is an older V8 benchmark which heavily uses objects. It is now ~30% faster and its Wasm binary is >4x smaller (402kb -> 92kb). Also, Test262 (featuring 50k test files) now runs twice as fast (~6m -> ~3m), thanks to not having to compile as many unused functions.

It also gives a small Test262 conformance boost, so 0 downsides! (+0.10%)

Thanks for reading! As always, you can follow progress and message me on [Twitter](https://x.com/CanadaHonk) or [Bluesky](https://bsky.app/profile/goose.icu) or [email](mailto:honk@goose.icu).