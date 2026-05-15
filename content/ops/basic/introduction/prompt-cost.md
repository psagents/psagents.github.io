# Why agent loops get expensive — and what the prompt cache does about it

People call it different things — *token cost*, *prompt cache*, *KV cache*, *context cache*. They're all pointing at the same thing: the meter that runs every time your agent takes another turn. This post is about why that meter exists, why it climbs faster than you'd expect, and what providers like Anthropic are actually pricing when they list a "cache read" line.

To get there we need a working definition of *agentic AI*. Not the most complete one — the minimal one.

## 1. The minimal definition

![What is Agentic AI?](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/prompt-cost/externals/01.png)

That's it for this post: **AI with tool use in a loop.** Six words. We'll spend the rest of the post taking that definition seriously and seeing what it implies for your invoice.

## 2. It fits in ~60 lines of Python

If you accept the six-word definition, you can build an agent in an afternoon with nothing but the standard library — no SDK, no framework, no agent toolkit.

![claude-code-mini.py — about 60 lines, pure stdlib](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/prompt-cost/externals/02.png)

The whole program is one file. It hand-rolls the JSON over `urllib`, exposes exactly one tool (`bash`), and lets the model loop until it stops asking for tool calls. The point of the slide isn't that you should read every character — it's that the *machinery* of an agent is small.  The machinery or scaffolding around the AI model itself is often referred to as **Harness** these days.

## 3. The loop, zoomed in

![The agent loop — call, detect, run, feed](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/prompt-cost/externals/03.png)

Four lines of intent, everything else is JSON over `urllib`:

1. **Call the model** with the **full message history**.
2. **Detect tool-use blocks** in the response. If there are none, return — the model is done.
3. **Run them in the shell.**
4. **Feed the results back** by appending them to `messages`, then loop.

The body of `agent_turn()` is essentially:

```python
def agent_turn():
    while True:
        resp = call_llm(messages)
        messages.append(resp)
        tools = [b for b in resp if b.tool_use]
        if not tools: return                        # model is done
        out = [run_tool(t.input.cmd) for t in tools]
        messages.append({"tool_result": out})
```

A single agent turn is *one* round trip: send `messages`, get a response, maybe execute some tools, collect their output (or errors). What makes it an *agent* — rather than a glorified chat completion — is that both the assistant's response and the tool outputs get appended back onto `messages` before the next iteration. The model's next decision is informed by everything that has happened so far. When the model stops asking for tools, the loop exits. That's the whole lifecycle.

## 4. The list that won't stop growing

![Filling up the context window](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/prompt-cost/externals/04.png)

Look at the two highlighted `messages.append(...)` lines. Every iteration appends *twice*: once for the assistant's response, once for the tool results. The list is monotonically growing. The context window fills up turn by turn — and unlike a human conversation, nothing in the loop ever forgets or summarizes. Whatever was in `messages` on turn 1 is still in `messages` on turn 20.

Innocent on its own. The cost story shows up on the very next line.

## 5. Every turn pays for every previous turn

![Submit the entire context — including tokens that have been processed before](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/prompt-cost/externals/05.png)

`call_llm(messages)` sends the **entire** `messages` list every time. The API is stateless — the server has no memory of your prior request, so the only way the model can reason over the conversation so far is for the client to re-ship the whole history.

Every token already in `messages` gets re-sent on every future turn. A long agent loop ends up paying for the same prefix again and again — that's the cost shape that makes agents different from one-shot chat completions. (To be precise: it's the *prompt* portion that gets re-billed each turn, not the completions. Output tokens are billed once, when generated.)

## 6. The prompt cache, priced

![Token (KV) Cache — Anthropic price table](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/prompt-cost/externals/06.png)

This is what providers do about it. Look at the Sonnet 4.6 row — three cells underlined:

- **Input tokens (fresh):** $3.00 per 1M
- **5-minute cache write:** $3.75 per 1M
- **Cache read:** $0.30 per 1M

Two things to notice. First, **cache reads are 1/10 the price of fresh input.** The second turn of an agent loop, if the prefix is cached, is paying ten cents on the dollar for everything the first turn already paid for. Second, **cache writes cost a premium** — about 25% more than a fresh token for the 5-minute TTL, and 2× ($6.00) for the 1-hour TTL. The cache isn't free; it's an investment.

A toy worked example. Suppose your agent runs for 10 turns, and the system prompt is a stable 5,000-token block at the front of every request — the same text gets re-sent on every turn:

```
prefix = 5,000 tokens
turns  = 10

No cache:
    5,000 * 10           tokens at $3.00 / 1M = $0.1500

With 5-minute cache:
    5,000        (write) at $3.75 / 1M       = $0.01875
  + 5,000 * 9    (read)  at $0.30 / 1M       = $0.01350
                                              --------
                                                $0.03225

Ratio: $0.1500 / $0.0323 ≈ 4.65× cheaper
```

And that's only the static head — real loops cache much more than that as the conversation accumulates.

The catch is the TTL — *time to live*, the window during which a cached prefix stays valid. After it expires, the next turn pays write rate again. The 5-minute TTL covers most interactive agents fine. But a single tool call that takes longer than the TTL — a slow build, a long sleep, a human-in-the-loop pause — can force a re-write of the prefix at write rate. Concrete case: you're mid-session with a coding agent, step away for an hour-long meeting, and come back to resume — the prefix has aged out, so the next turn re-pays write rate on tokens that *were* cached when you walked away. The 1-hour TTL exists for exactly that case, at the cost of a steeper write premium.

## So what

The minimal definition — *AI with tool use in a loop* — is enough to explain why agentic AI has its own cost shape. The loop is what makes the system useful (it gets to react to what its tools returned), and the loop is also what makes the bill keep climbing (every prior turn rides along on every future request). Prompt caching is the lever that flattens that curve, but it has its own pricing geometry: cheap reads, premium writes, TTLs that matter.

If you're building an agent, the practical implication is: **keep the prefix stable**, structure your `messages` so the cacheable part comes first, and pay attention to how long your tool calls actually take. The wire protocol stays visible in 60 lines of Python — and so does the receipt.
