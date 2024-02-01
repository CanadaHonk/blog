---
title: "Porffor: Halving ASCII string memory usage with ByteString"
description: "My JS engine has a new string representation, here's what/why/how."
date: 2024-01-31
draft: false
---

While doing Node contributions on and off, I kept spotting that [V8 has *a lot* of different string types](https://github.com/v8/v8/blob/941b945b/src/objects/objects.h#L134-L151). This has stuck in the back of my head for a while, until coming back yesterday.

In JS, strings are UTF-16, and this is quite heavily ingrained into the language. Most engines use UTF-8 internally and convert between when needed for `length`, etc. Porffor however uses UTF-16 internally completely.

For the purposes of this post, the main difference is that every UTF-16 character is at least two bytes, while every UTF-8 character is at least one byte. For example, to represent the character `a`:
- UTF-16: `0x61 0x00`
- UTF-8: `0x61`

You can clearly see UTF-16 is essentially wasting a byte of memory for just one character, the upper byte is just unused for ~ASCII/Latin-1 values (char codes below 256). This means when given a plain string like `'abcdef'`, Porffor is using almost 2x the memory required. This is why I have now added `ByteString`!

`ByteString` is a new string representation which is used when the compiler detects strings which only use the lower byte of UTF-16. This is possible as strings are immutable in JS, so we know what the possible range of character codes are at compile-time.

Let's compare the memory of the string `'abcdefghijklmnopqrstuvwxyz'` in Porffor's regular `String` vs `ByteString`:

```
String     | 1a 00 00 00 61 00 62 00 63 00 64 00 65 00 66 00 67 00 68 00 69 00 6a 00 6b 00 6c 00 6d 00 ...
ByteString | 1a 00 00 00 61 62 63 64 65 66 67 68 69 6a 6b 6c 6d 6e 6f 70 71 72 73 74 75 76 77 78 79 7a
```

You can see both start with 4 bytes for the length as an integer, then differ dramatically. `String` is clearly using two bytes for each character, leaving a wasted `0x00` for each character. `ByteString` is just using the lower byte, so each character only uses a single byte, ~halving the memory usage!

Thanks for reading! This is behind a flag (`-bytestring`) for now due to some inter-string-type trickiness, this was a proof of concept to see if it is feasible and has benefits, which it clearly does :)