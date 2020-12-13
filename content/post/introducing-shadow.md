---
title: "Introducing Shadow"
description: "Shadow is a new novel browser engine made almost entirely in JS"
date: 2023-10-27
draft: false
---

So I started making a browser engine (for fun) a few days ago, it felt kind of inevitable [after starting a JS engine](https://porffor.goose.icu), so here we are. Here's a short rundown. [Source code too!](https://github.com/canadahonk/shadow)

# [Try it in your browser!](https://shadow.goose.icu)

#### Screenshot of Shadow's welcome page running inside Shadow (as of writing)
![Screenshot of Shadow's welcome page in shadow](https://github.com/CanadaHonk/shadow/assets/19228318/9431b624-94aa-4209-87c9-60d2478badf6)

### What?

A browser(/web) engine essentially takes in a URL(/etc) and gives you it rendered into a window for you to view and interact with. Shadow does this too, almost entirely from scratch, made in JS. It runs in your browser! Node backend soon™ too? The host browser(/etc) is only used for networking (`fetch`) and renderer backend (`<canvas>`).

#### Components of Shadow

![Component flowchart](https://mermaid.ink/svg/pako:eNpVkT1vhDAMhv8KygxVaTuB1Kljp96axSLukQtJUByK0N399zogFJAyOI_f1x_yXXReoWjENcDYF98_rXRF0YH7A6peqs_eU0zEYZx9MNpdj5R6UH6uGEQ7jBAIwwlnV8LKW2YdUVZmH2c4vzbfBYwGWPwUszn_t4hRQKcwbOX2-KQ8wG2vBGcgmzahUXPSemdwOXKjkXDIjR9-jJV2j9vqvlEed42TTbp1Lu3MJS4DFq9lXdb83gqKwRtsAqr2LKnfy_pjT_MFllaUwvI8oBXf5J7UUsQeLUrRcKggGCmke7IOpugvi-tEE8OEpZhGBRG_NHAhK5pfGAif_zYlpas?bgColor=101418)
(red = external/not me)

### Why make it?

It's just for fun, no really. Learning too. It will probably never work with 90% of websites, and that's okay. ~~Also it's funny to see people react in many forms of "wtf?"~~

### Screenshot of serenityos.org running inside Shadow (left) vs Firefox (right)
<p>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/2f9c14e9-a23d-4f92-9f7d-d29aa68ba1bf" width=600>
<img src="https://github.com/CanadaHonk/shadow/assets/19228318/19850ac0-2950-4919-8147-a957eefa7c09" width=600>
</p>

Pretty spot on! No list markers yet. Colors are different as UA/browser defined.

### Name

As with all my recent projects, the name is because I thought it was kind of funny at the time. Shadow is named after the defunct &lt;shadow&gt; element. I mostly stole this idea from Blink (was it intentional to name it after a dead HTML element at the time?). Also it sounds spooky and mysterious so that's a bonus I guess?

### It supports JavaScript??!

Yes, kind of. This is quite complicated (as you can probably imagine), so I'll do it in a separate future dedicated post if you're interested (DM/reply me on twitter).

### Why publish it?

Why not? If someone can learn something or just find what I make fun/interesting, that makes me happy :)

### But making a new browser engine is impossible!

[No](https://ladybird.dev). [It](https://servo.org). [Isn't](https://www.ekioh.com/flow-browser/)! *(Also I don't really care how possible/feasible something is.)*