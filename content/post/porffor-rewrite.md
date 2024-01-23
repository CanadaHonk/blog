---
title: "Porffor: v0.2 rewrite done!"
description: "My JS engine's months-long rewrite is finally done!"
date: 2024-01-22
draft: false
---

My JS engine [Porffor](https://porffor.goose.icu)'s rewrite is finally complete! If you follow me, you might have seen me ~~ranting~~ talking about it over the past few months. I eventually gave up on it about a month ago but resumed a few days ago, and now it is done.

## Type system

Previously, the compiler inferred the types of variables/etc at compile-time. If you did:
```js
// always gives 1 in Porffor v0.1
Math.random() > 0.5 ? 1 : true
```
It would (inaccurately!) always presume the outcome was a number (`typeof ... // number`) as it cannot know at compile-time. This was the main limitation with Porffor, as types are quite important for JS, as you can imagine.

Porffor v0.2 *(I have begun versioning major milestones)* has a new ✨runtime type system✨ which had to be done in a [single massive rewrite](https://github.com/CanadaHonk/porffor/commit/77e30e836b63e81c01f11077bc724e534101cada). It had to be a single commit/push as I had to entirely remove the old type system (breaking everything which used it before) and rewrite everything which had been using it for the new type system. In the end, the diff was **+1532 -950**. Considering the compiler is ~4k lines in total, quite huge!

Also, by the time I pushed it, it had a positive test262 (~JS cross-engine test suite) score improvement compared to pre-rewrite! This means there aren't (many) regressions with it. Quite a few tests before were incorrectly passing when they should have been failing due to bad type information, so I don't mind breaking them :^)

## Pluggable parser

As a bonus, Porffor's parser is now pluggable! This means you can pick the parser package you want to use as a compile-time option, eg:

```sh
porf -parser=acorn # use acorn for parsing
porf -parser=@babel/parser # use @babel/parser for parsing
```

Hopefully in the future this will allow Porffor to parse non-standard JavaScript; allowing implementing proposals with Babel's parsing, and possibly... something else secret for now... stay tuned ;)

## IEEE 754

As a final note, I also rewrote how Porffor implements IEEE 754 (floating point numbers) encoding/decoding. It now uses JS typed arrays instead of using a 3rd party implementation in JS. This satisfyingly results in encoding and decoding being only **2 lines**, compared to >100 before, and the 2 are quite simple:

```js
export const ieee754_binary64 = value => [...new Uint8Array(new Float64Array([ value ]).buffer)];
export const read_ieee754_binary64 = buffer => new Float64Array(new Uint8Array(buffer).buffer)[0];
```

Thanks for reading! You can [try out Porffor in a web playground](https://porffor.goose.icu/) (also refreshed the site a bit, still WIP). If you have ideas of JS features I should implement next or (simple!) projects I should try to get working you can reach out as always :)