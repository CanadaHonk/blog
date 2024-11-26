---
title: "Optimizing this in Porffor"
description: "this can be surprisingly bad for performance, especially binary size, due to one design flaw."
date: 2024-11-26
draft: false
---

`this` in JS can be difficult for performance due to one design flaw which strict mode (`"use strict"`) actually fixes; `this` is `globalThis` in non-constructors by default:

```js
function sloppy() {
  return this;
}
sloppy(); // globalThis

function strict() {
  "use strict";
  return this;
}
strict(); // undefined
```

Due to this, in some situations Porffor has to presume the worst and include all of `globalThis` for some code which looks like it shouldn't be that bad:

```js
function foo() {
  console.log(this.wow);
}

foo();
```

Including `globalThis` is bad news in ahead-of-time JS engines like [Porffor](https://porffor.dev), as it forces the compiler to include essentially every part of the JS engine/runtime, outright killing some optimizations. However, there are some static analysis tricks Porffor does to help optimize most scenarios:

### Strict mode

Strict mode fixes this by making `this` undefined for these cases instead of `globalThis`, so we can just skip this entire problem if we are compiling in strict mode!

### Lazy `this` as `globalThis`

#### Compile-time

`this` is lazily made both at compile-time and runtime. Unless you use `this` somewhere in a function, we can just not do anything for it at all.

#### Runtime

With the following code:

```js
function foo() {
  if (Math.random() < 0.1) {
    this.someAccess;
  } else {
    // noop
  }
}

foo();
```

`this` never gets "made" into `globalThis` until you actually access it, so there is no wasted overhead for every single call if it only rarely gets used.

### Constructors

Functions only called as constructors (`new ...`) never have `this` as `globalThis`, so if we know some functions are only constructed, we can just never have to account for it:

```js
function Foo() {
  this.prop = 'value';
}

new Foo();
```

`Foo` is never referenced or called outside of `new ...` so it must only be constructed! However, if we have an indirect reference to `Foo` we have to abort this optimization as that reference could be used for anything:

```js
const indirect = Foo;
indirect(); // not constructed, this should be globalThis!
```

### Prototype methods

```js
function Foo() {
  this.prop = 'value';
}

Foo.prototype.func = function () { this.prop = 'new value'; };

new Foo();
```

Previously we would abort from this optimization in both the constructor (`Foo`) as it is referenced; and the prototype method (`Foo.prototype.func`) as the member assignment is using a reference to the function. Happily, a new optimization allows us to fix this when seeing the pattern `Foo.prototype.prop = `! This optimization made Porffor's slowest benchmark 15% faster! But more importantly, the Wasm binary output for it is now over 3x smaller, as it is no longer unnecessarily including all of `globalThis`:

```sh
~/porffor$ ./porf bench/richards.js # old, perf test, lower is better
708
~/porffor$ ./porf wasm bench/richards.js richards.wasm; du -h richards.wasm # old, wasm size test, lower is better
424K    richards.wasm
~/porffor$ git stash pop -q
~/porffor$ ./porf bench/richards.js # new, perf test, lower is better
600
~/porffor$ ./porf wasm bench/richards.js richards.wasm; du -h richards.wasm # new, wasm size test, lower is better
116K    richards.wasm
```

Yes, it was still under 500KB of Wasm even with all of `globalThis` ;). Thanks for reading! You can follow progress and message me on [Twitter](https://x.com/CanadaHonk) or [Bluesky](https://bsky.app/profile/goose.icu) (or [email](mailto:honk@goose.icu)).