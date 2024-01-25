---
title: "Porffor: Type annotations for performance"
description: "My JS engine can now parse TS and use it for optimizations!"
date: 2024-01-25
draft: false
---

My JS engine [Porffor](https://porffor.goose.icu) can now parse TypeScript, as I added pluggable parser support which includes Babel's parser (which can parse TS). This itself isn't that interesting, however I can now use those type annotations as compiler hints to optimize!

## Basic example

```js
let a = 1;
if (a) {
  console.log(typeof a);
}
```

For the above JS, Porffor has to do several checks of the type of `a`:
1. `if (a) {`
  - If string, true if non-blank (not `''`)
  - If undefined, false
  - ...
  - **If number, true if not zero**
2. `typeof a`
  - If string, `'string'`
  - If object, `'object'`
  - ...
  - **If number, `'number'`**

I already optimize out types which are completely unused in the file. For example, all the string checks are removed if the file has 0 strings used in it. But if there were any strings, all string checks remain. Porffor could try to infer the type automatically but this is tricky and could break. JIT engines can sometimes do this by presuming the type of something once it has been ran with that type several times, etc. But since we are compiling AOT and not JIT, we cannot de-opt/undo that guess if it becomes wrong - instead we would just crash.

But we could define the type using type annotations (TS)!

```ts
let a: number = 1;
if (a) {
  console.log(typeof a);
}
```

Now we have to do **0 checks of the type of `a`**, since we already know it at compile-time. This can allow for some big speedups :)

## Progressive types

Porffor does not type check itself. You can use your own (with your own config too) if you want that. The input also does not have to be completely typed, allowing "progressive types". Types which exist can help speed things up, and untyped things will just be treated regularly.

## Benchmark

I'm using [a basic Brainf... interpreter I wrote in JS (`bf.js`)](https://github.com/CanadaHonk/porffor/blob/main/bench/bf.js), plus [a typed version (`bf.ts`)](https://github.com/CanadaHonk/porffor/blob/main/bench/bf.ts). Here are the times running BF which draws the Mandelbrot set, with different commands (`porf` = Porffor):

#### `porf bf.js`: 272s
This just runs the JS file regularly, with default options.

#### `porf bf.ts -parse-types`: 274s
This runs the TS file, but only parses the types without using them for optimization. About the same time as regular JS, within error.

#### `porf bf.ts -parse-types -opt-types`: 184s
`-opt-types` tells Porffor to use the type annotations as compiler hints. 1.5x speedup, just from using types!

#### `porf bf.ts -parse-types -opt-types -valtype=i32`: 84s
`-valtype=i32` also tells Porffor to use `i32`s (32 bit integers) as `number` instead of `f64`s. In the future I plan to add a special type to do this instead of changing `number` for everything, but the compiler isn't that smart yet heh. Another 2.2x speedup compared to last, or 3.2x in total!

#### `porf bf.ts -parse-types -opt-types -valtype=i32 -funsafe-zero-proto-checks=charcodeat`: 67s
`-funsafe-zero-proto-checks=charcodeat` additionally informs the compiler to do no input out of bounds/etc checks for `String.prototype.charCodeAt`. In the future, this should also be able to be automatically detected and done for you. Mild 1.3x speedup compared to previously, resulting in 4x compared to original.

#### `node bf.js`: 66s
For comparison, Node running the same JS file. Obviously, Node is JIT compiled so has a big advantage compared to compiling AOT to bytecode. Yet, we still roughly match it with the right options! 🚀

## Conclusion

The best part of this for me is that [the main diff for this feature was small, only +131 -25](https://github.com/CanadaHonk/porffor/commit/a6a92e01ac1e09383f5c3f24e55f2648ff714b7a), and only took me ~an evening to do; thanks to the great foundation from [the rewrite](/porffor-rewrite/). Please note both regular JS and typed will likely speedup even more in the near future! This is just today. I haven't compared to many other engines as I'm saving that for a later post when I've done more benchmarking and testing.

Also for comedic relief, this makes the full description of Porffor even more complicated:
> A from-scratch experimental AOT optimizing JS/TS -> Wasm/C engine/compiler/runtime in JS

Thanks for reading! :)