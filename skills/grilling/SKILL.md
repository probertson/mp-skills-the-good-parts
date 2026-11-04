---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree **one question at a time**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. From the frontier, ask the single question whose answer unblocks the most, and give your recommended answer. Then wait for the user's answer before asking anything else. Keep the rest of the frontier to yourself; don't preview upcoming questions.

Format a question like so:

```
❓ **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer. The ❓ block is the last thing in your message: no summary, preamble or upcoming questions after it.

Each answer reshapes the tree: a settled decision pushes the frontier outward and unblocks questions that depended on it. Recompute the frontier and ask the next question. The user may answer with a question or a longer reply of their own; follow that thread until it settles, then return to the frontier.

Finding _facts_ is your job, never the user's. When the next question needs a fact from the environment (filesystem, tools, etc.), find it before asking; you may dispatch a sub-agent with a fresh context window to do it. Don't ask the user for anything you could look up yourself. **Facts first, then ask**: never ask a question while a lookup that could change it is still running. Wait for the result, recompute the frontier, then ask. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
