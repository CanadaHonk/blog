---
title: "Porffor FFI"
description: "Porffor now has experimental FFI support!"
date: 2024-08-15
draft: false
---

[FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (foreign function interface) allows JavaScript inside JS runtimes to call external native shared libraries, to do things which would be otherwise difficult to do in JS. As of today, Porffor now has (early/experimental) support for it! It is currently limited to the native/C target in this early version. To test and benchmark, I adapted [a benchmark](https://github.com/littledivy/blazing-fast-ffi-talk/blob/main/sqlite.deno.ts) by [Divy Srivastava](https://littledivy.com) which uses [`sqlite3`](https://www.sqlite.org/cintro.html). Porffor's `dlopen` API intentionally has ~identical arguments to Deno, only differing slightly with the return value, looking like:

```js
const {
  sqlite3_initialize,
  sqlite3_open_v2,
  sqlite3_exec,
  sqlite3_prepare_v2,
  sqlite3_reset,
  sqlite3_step,
  sqlite3_column_int,
} = Porffor.dlopen("libsqlite3.so.0", {
  sqlite3_initialize: {
    parameters: [],
    result: "i32",
  },
  sqlite3_open_v2: {
    parameters: [
      "buffer", // const char *filename
      "buffer", // sqlite3 **ppDb
      "i32", // int flags
      "pointer", // const char *zVfs
    ],
    result: "i32",
  },
  // ...
```

<img src="https://github.com/user-attachments/assets/7ece07d4-90db-436a-80b3-69b238874ee1" width=600 style="filter: brightness(0.9); float: right; margin: 0 var(--card-padding)">

...and the benchmark results are very promising! Around 10% faster than Deno, which has [a lot of its own tech just to optimize FFI](https://littledivy.com/turbocall.html) and is well regarded for its FFI performance. It is also ~30% faster than Bun, which has decent FFI performance. Node does not really have built-in FFI so is not compared here, but it would likely be slower than Bun.

The benchmark measures FFI performance by running SQL queries in `sqlite3`, as there are a lot of FFI calls and not much JS work (for integers), so it demos FFI overhead nicely while being more realistic by using a popular library, in my findings and opinion.

Thanks for reading! You can [follow progress](https://x.com/CanadaHonk) and [message](https://x.com/CanadaHonk) or [email](mailto:honk@goose.icu) me in these places :^)

<div style="clear: both"></div>