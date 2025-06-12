---
title: "Dissecting Dia"
date: 2025-06-11
draft: false
tangent: true
---

Whether you like it or not, Dia presents a relatively novel browser user experience and is now in a publicly available beta, so I think it is worth looking into how it works. Here are some interesting day 1 findings from analyzing its binaries:

## Models

Dia has the capability to use both locally ran and hosted models, although I'm not sure if/when the former is used. Hosted models mentioned internally include various providers (OpenAI, Google, Anthropic) but also seems to focus on OpenAI/GPT specifically: they have fine-tuned GPT-4.1, 4o, 4o-mini. My guess is they use 4.1 or 4o or 4o-mini as the main model available but there are also mentions of o1-mini, o1-preview and o3-mini so... who knows.


## Tool calls

Dia, unsurprisingly, uses tool calls with its LLM usage. However, the tools available are... interesting:

### Calculator

This seemingly boring tool allows the model to do mathematical computations; but it is implemented in an amusing way: **by just evaluating JS expressions in a "sandbox"**.

Straight from the internals:

> **Calculator**<br>
> Purpose: Perform mathematical calculations.

> Properties:
>   - A JS ES6 expression that will be evaluated using JSCore, e.g. `Math.sqrt(4 * 1.01)`.

> Use When:
>   - Mathematical computation is required.
>   - You can reasonably create a mathematical expression to calculate the result, either based on your own knowledge, context provided in the query, or results of a web search.
>   - The generated expression must be compatible with JavaScript syntax and should only use JavaScript-compatible functions and operators.

> Constraints:
>   - Avoid functions and syntax unsupported by JavaScript (e.g., `DATE()`, `INTERVAL`, custom date arithmetic, or unsupported math functions like `sqrt()`).
>   - Use `Math` functions as needed (e.g., `Math.sqrt()`, `Math.pow()`).
>   - Ensure any exponentiation uses `Math.pow()` or the `**` operator.


I say "sandbox" in quotes as it is implemented with this:

```js
(function(userScript) {
  'use strict';
  // Save reference to the original Math
  const originalMath = Math;

  // Clean the global object to remove access to everything
  for (const key in globalThis) {
    if (key !== 'Math') {
      try {
        delete globalThis[key];
      } catch (e) {
        globalThis[key] = undefined;
      }
    }
  }

  // Make sure Math is frozen to prevent modifications
  Object.freeze(Math);

  // Add minimal undefined for JavaScript semantics
  Object.defineProperty(globalThis, 'undefined', {
    value: undefined,
    writable: false,
    configurable: false
  });

  // Prevent adding properties to globalThis
  Object.freeze(globalThis);

  // Evaluate the user script with only Math available
  try {
    return (new Function('return (' + userScript + ');'))();
  } catch (error) {
    throw new Error('Evaluation failed: ' + error.message);
  }
})(...)
```

For the purposes of this tool call it is *probably* fine, but I am 90% sure you could bypass that if you truly tried.


### Unusable Web Attachment

This tool is dedicated to indicating the user's query cannot be answered with the page data scraped by Dia.

Straight from the internals:

> **UnusableWebAttachment**<br>
> Purpose: Indicate that the user's query cannot be answered because the scraping of a web page (current-webpage or referenced-webpage) did not capture required information.

> Properties:
>   - The URL of the flagged web page
>   - A concise explanation of what is wrong with the scraped web page#

> Rules:
>   - Prioritize calling this function if a scraped web page is missing expected content needed for answering the user's query.

> Examples of reasons for calling this function:
>   - The scraped content is empty or nearly empty.
>   - Some content was scraped, but the parts required to answer the user's query are missing.
>   - The scraped content appears to have been truncated in a way that excluded parts required to answer the user's query.
>   - The scraped content appears to be corrupted or inintelligible.


### Web Search

This tool allows the model to search the web. Interestingly, it specifically specifies Google as the search engine, as seen below.

> **WebSearch**<br>
> Purpose: Fetch up-to-date, specific, or contextual information that may not be stable or broadly known.

> Properties:
>   - An expertly-written, precise Google search query that would provide the info you need. When searching location-based things, include the name of the user's town, city or neighborhood.

> Use When:
>   - Time-Sensitive Information: The request involves data that changes frequently
>   - Local or Context-Specific Details: The user requests information tied to a particular location, event, or condition
>   - Emerging or Fast-Evolving Topics: For areas with rapid advancements, such as AI model updates, new technology releases, or scientific breakthroughs
>   - Verification of Potentially Outdated Information: Confirm or update facts that are likely to change over time.
>   - Clarification on Conflicting Information: When multiple perspectives exist on a topic, use search_web to gather recent, reputable viewpoints

> Do NOT use when the user is asking a research-type question focusing on historical or well-established topics; in this case Dia should rely on its own knowledge unless specifically asked for recent developments.


### Final notes

It is also nice to see that Dia is using the latest version of Chromium as of release/writing (137), although we'll see how quick they are to maintain that.

I'm intentionally missing my personal opinions on Dia/Arc/The Browser Company as I am honestly not sure how to formulate them, this isn't meant to be positive or negative in any form, rather noting interesting parts of it (maybe I'll do a follow-up with real opinions one day).
