# The Two-Call Claude Tool-Use Pattern

**Date:** 2026-06-29
**Category:** Agentic AI / Claude API / JavaScript

## What I Did
Walked through `index.js` in my `inventory-agent` project line by line.
This is the agent that connects to my vehicle inventory REST API and lets
Claude answer natural language questions about the inventory — like
"do you have any trucks available?"

The goal was to understand the two-call pattern that powers the whole thing.
I built this during the Agentic AI Bootcamp but honestly the code felt like
a black box. Today I actually broke it down and got it.

## What Was Confusing

### 1. return new Promise((resolve, reject))
I'd never seen this before — my intro JS course at community college only
covered synchronous code where everything runs top to bottom in order.

Promises are JavaScript's way of handling things that take time, like an
API call over a network. You can't just freeze the program and wait, so
instead you get back a Promise — basically a buzzer that goes off when
the data is ready.

- `resolve` fires when it succeeds — "here's your data"
- `reject` fires when it fails — "something broke"
- `await` is the cleaner way to wait for a Promise to resolve before
  moving to the next line

### 2. Why Two Calls?
I wasn't clear on why you needed to call Claude twice. Felt redundant.

Turns out the two calls do completely different jobs:
- **Call 1** — Claude reads the question and decides if it needs a tool
- **Call 2** — Claude gets the real data back and writes the actual answer

They can't be combined because Claude doesn't have the data yet on the
first call. You have to go get it yourself in between.

### 3. Claude Can't Call the API Directly
This one surprised me. Claude doesn't reach out to your inventory API —
it just tells YOU what it needs. Your code does the actual API call,
then hands the results back to Claude.

Claude is the brain. Your code is the hands.

## What I Now Understand

**The full two-call flow:**
Send question + tool menu  →
←    "I need search_inventory({ make: 'truck' })"
Call inventory API yourself →
Send results back          →
←    "Yes, you have 3 trucks available..."


**Promises in one sentence:**
`new Promise` is JavaScript's way of saying "this will take a moment —
I'll let you know when it's done." `resolve/reject` signal whether it
worked or not. `async/await` is cleaner syntax for the same idea.

**The message history on the second call matters:**
You're not just sending the results — you're sending the full conversation:
original question + Claude's first response + the tool results. Claude
needs that context to write a coherent answer.

## Key Code — The Second Call
```javascript
const finalResponse = await client.messages.create({
    messages: [
        ...messages,                              // original question
        { role: "assistant", content: response.content }, // call 1 response
        {
            role: "user",
            content: [
                {
                    type: "tool_result",
                    tool_use_id: toolUse.id,      // ties result to request
                    content: JSON.stringify(results), // actual API data
                }
            ]
        }
    ]
});
```

## Takeaway
Coming in with only intro JS, the async patterns and the two-call
structure were the hardest parts. Understanding that Claude is the
decision-maker but your code does the actual work was the unlock.
This pattern will show up everywhere in agentic AI development.
