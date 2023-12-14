---
title: "porffor: the value wrapping dilemma"
description: "my JS engine and why it is currently terrible"
date: 2023-12-14
draft: false
---

this is informal and rambly, hence *lowercase*.

so my JS engine, [porffor](https://porffor.goose.icu), takes in JS and outputs Wasm, plus is written in JS. if you don't know it already, you might be thinking "what??". I'm going to skip over that for now.

wrapping JS values. JSC/SM/LibJS use NaN boxing. NaN boxing is where you store non-number values as NaN (wow) and use the lower bits to store additional data for the value, like a type and pointer. V8 uses [pointer tagging](https://en.wikipedia.org/wiki/Tagged_pointer). I don't know how pointer tagging works, [read this](https://stackoverflow.com/questions/63550957/why-does-v8-uses-pointer-tagging-and-not-nan-boxing).

how does porffor wrap JS values? the answer is it doesn't (yet). porffor uses a single type for all variables, by default f64 because JS numbers. so `let a = 0` makes a variable referred to as `a`, and it is just the number stored. but what if we do `let a = []`. now the number actually stored in the variable referred to as `a` is the pointer to the array in memory. what's the problem? we need type information.

why do we need type information? say we have a function, `foo = x => x.length`. when generating the (Wasm) bytecode for this function, we have no clue what `x` is. a number? a string? an array? we need to know what it is to know what bytecode to generate. currently, porffor just presumes all variables are numbers unless otherwise known so this example basically does not work.

so what should we do? we need to wrap values. or do we? so far I have settled on 3 approaches, in increasing insanity:
1. NaN boxing. just box all our non-number values with NaN boxing. upside: traditional, well documented. downside: bytecode is more complex, worse perf.
2. make 2 Wasm locals/vars for each JS variable, the value/pointer and type separately. upside: simple bytecode, good perf? downside: more arguments/etc everywhere.
3. procedurally generate function variants depending on types of arguments (inferred at compile-time). oh yeah. upside: best perf, simplest bytecode. downside: binary size. generally insane.

I'll probably go for approach 2 and starting working on the engine again soon, probably over the holidays. this is the main limitation of the engine so far, so really needs addressing for the future.

thanks for reading! I wrote this on a whim at 1am as I've had this problem stuck in my head for months and thought it might be interesting enough to share. if you genuinely liked this, tweet/toot me and I'll probably write more like it.