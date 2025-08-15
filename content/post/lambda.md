---
title: "Eliminating JavaScript cold starts on AWS Lambda"
description: "Porffor can run on Lambda now!"
date: 2025-08-14
draft: false
---

## How? Enter Porffor

[Porffor](https://porffor.dev) is my JS engine/runtime that compiles JavaScript ahead-of-time to WebAssembly and native binaries. What does that actually mean? You can compile JS files to tiny (<1MB), fast (millisecond-level) binaries:

```sh
~$ bat hi.js
─────┬──────────────────────────────────────
   1 │ console.log("hello blog!")
─────┴──────────────────────────────────────
~$ porf native hi.js hi
[271ms] compiled hi.js -> hi (12.9KB)
~$ du -h hi
16K     hi
~$ ./hi
hello blog!
```

Node and Bun offer "compile" options, but they bundle their runtime with your JS rather than actually compiling it as if it was C++ or Rust. Porffor does that, allowing for much smaller and faster binaries:

```sh
~$ deno compile -o hi_deno hi.js
~$ bun build --compile --outfile=hi_bun hi.js
~$ du -h hi*
16K     hi
97M     hi_bun
82M     hi_deno
4.0K    hi.js
~$ hyperfine -N "./hi" "./hi_deno" "./hi_bun" --warmup 5
Benchmark 1: ./hi
  Time (mean ± σ):     631.4 µs ± 128.5 µs    [User: 294.5 µs, System: 253.1 µs]
  Range (min … max):   465.3 µs … 1701.3 µs    2762 runs

Benchmark 2: ./hi_deno
  Time (mean ± σ):      37.4 ms ±   1.7 ms    [User: 22.5 ms, System: 16.0 ms]
  Range (min … max):    33.8 ms …  42.2 ms    74 runs

Benchmark 3: ./hi_bun
  Time (mean ± σ):      15.9 ms ±   1.2 ms    [User: 8.7 ms, System: 9.6 ms]
  Range (min … max):    13.7 ms …  19.2 ms    175 runs

Summary
  ./hi ran
   25.24 ± 5.50 times faster than ./hi_bun
   59.30 ± 12.36 times faster than ./hi_deno
```

What's the trade-off? You have to re-invent the JS engine (and runtime) so it is still very early: limited JS support ([but over 60% there](https://porffor.dev/#test262)) and currently no good I/O or Node compat (yet). But, we can use these tiny fast native binaries on Lambda!

<br>

## Lambda

A few days ago I got Porffor working on Lambda, not simulated locally but really hosted on AWS! I wrote a cold start benchmark for Node, [LLRT](https://github.com/awslabs/llrt) (Amazon's own experimental JS runtime optimizing cold starts) and Porffor running identical code:

```js
export const handler = async () => {
  return {
    statusCode: 200,
    headers: { "Content-Type": "text/plain" },
    body: "Hello from " + navigator.userAgent + " at " + Date()
  };
};
```

Since we're benchmarking cold start, the workload does not matter as we are interested in just how we are running here (for context most Lambdas run for <1s, typically <500ms). I spent over a day just benchmarking and even with my biases, the results surprised me.

### Node
<img alt="A graph of benchmark results for Node, explained below" src="https://raw.githubusercontent.com/CanadaHonk/porffor/refs/heads/main/bench/lambda/node.png" style="filter: brightness(0.8)">

Here is Node (managed, `nodejs22.x`), our main comparison and baseline. Surprisingly alright, but still far from ideal: having your users have to wait up to 0.3s due to a technical limitation out of your control just sucks.

We don't prioritize memory usage here, as AWS bills based on allocated memory rather than actual usage. In this benchmark, we allocate the minimum (128MB), ensuring it remains below that threshold. I'll show the cost in GB-seconds, calculated as billed duration (from AWS) multiplied by allocated memory.

Also, Node is a managed runtime, meaning AWS supplies it for you. This significantly aids with initialization duration by allowing for effective caching. Crucially, we are not billed for this init duration, which profoundly impacts cost. (While an [AWS blog post](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/) indicates that this will change starting August 1st, this data is from August and does not yet reflect such charges. I will update if this changes.)

### LLRT
<img alt="A graph of benchmark results for LLRT, explained below" src="https://raw.githubusercontent.com/CanadaHonk/porffor/refs/heads/main/bench/lambda/llrt.png" style="filter: brightness(0.8)">

LLRT is ~3x faster than Node here, great! Unfortunately in my testing, it also costs ~1.6x more than Node. This is only due to the managed runtime trick explained before. This should change when they charge for that init or create a managed runtime once LLRT is stable. Overall, ignoring that hitch, much better than Node for this benchmark!

### Porffor
<img alt="A graph of benchmark results for Porffor, explained below" src="https://raw.githubusercontent.com/CanadaHonk/porffor/refs/heads/main/bench/lambda/porffor.png" style="filter: brightness(0.8)">

Porffor is ~12x faster than Node and almost 4x faster than LLRT in this case. Plus, even with Node's managed runtime trick, it is over 2x cheaper than Node (and almost 4x cheaper than LLRT). <sup>🫳🎤</sup> I hope this shows that when Porffor works, it works extremely well: Porffor's P99 is faster than both LLRT's and Node's P50.

<br>

## Conclusion

You might be expecting me to start shilling for you to plug Porffor into your Lambda instantly... but unfortunately not. Porffor is still very (pre-alpha) early.

Although, if you/your company is have small Lambdas (ideally no Node APIs) and want a free quick look on if Porffor could help you, please [email me](mailto:honk@goose.icu)! Porffor is actively improving and more code is working everyday.