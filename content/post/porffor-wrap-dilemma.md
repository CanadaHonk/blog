---
title: "Porffor: The value wrapping dilemma"
description: "My JS engine and why it is currently terrible"
date: 2023-12-14
draft: false
---

So my JS engine, [Porffor](https://porffor.goose.icu), takes in JS and outputs Wasm, plus is written in JS. If you don't know it already, you might be thinking "What?!". I'm going to skip over that for now.

Wrapping JS values. JSC/SM/LibJS use NaN boxing. NaN boxing is where you store non-number values as NaN and use the lower bits to store additional data for the value, like a type and pointer. V8 uses [pointer tagging](https://en.wikipedia.org/wiki/Tagged_pointer). I don't know how pointer tagging works, [read this](https://stackoverflow.com/questions/63550957/why-does-v8-uses-pointer-tagging-and-not-nan-boxing).

How does Porffor wrap JS values? The answer is it doesn't (yet). Porffor uses a single type for all variables, by default f64 because JS numbers. So `let a = 0` makes a variable referred to as `a`, and it is just the number stored. But what if we do `let a = []`? Now the number actually stored in the variable referred to as `a` is the pointer to the array in memory. What's the problem with this? We need type information.

Why do we need type information? Say we have a function, `foo = x => x.length`. When generating the (Wasm) bytecode for this function, we have no clue what `x` is. A number? A string? An array? We need to know what it is to know what bytecode to generate. Currently, Porffor just presumes all variables are numbers unless otherwise known so this example basically does not work.

So what should we do? We need to wrap values... or do we? So far I have settled on 3 approaches, in increasing insanity:
1. NaN boxing. Just box all our non-number values with NaN boxing. Upside: traditional, well documented. Downside: bytecode is more complex, worse perf.
2. Make 2 Wasm locals/vars for each JS variable, the value/pointer and type separately. Upside: simple bytecode, good perf? Downside: more arguments/etc everywhere.
3. Procedurally generate function variants depending on types of arguments (inferred at compile-time). Upside: best perf, simplest bytecode. Downside: binary size, generally insane.

I'll probably go for approach 2 and starting working on the engine again soon, probably over the holidays. This is the main limitation of the engine so far, so really needs addressing for the future.

Thanks for reading! I wrote this on a whim at 1am as I've had this problem stuck in my head for months and thought it might be interesting enough to share. If you genuinely liked this, tweet/toot me and I'll probably write more like it. :)
