# Context is the only lever

When you work with a coding agent, there is exactly one thing you control: what goes into its context window. Everything else is downstream of that. Skeptic or seasoned, this is the lever worth learning to pull well.

This post is the written version of a talk I gave for LCLS scientists and engineers. One idea, three ways it fails, and a glimpse of where the craft goes once it scales.

---

## The one idea

Strip away the magic and an agent is simple: context goes in, one next step comes out.

People assume it remembers. It doesn't. Every step is a fresh, stateless call, and nothing carries over that isn't in the window — it's closer to the character in *Memento* than to a colleague. It's a function, not a person, and once you see it that way the rest follows.

If it's a function, the output is only as good as what you feed in. So you're not chatting with it; you're engineering its input. Get the right material in, with room to spare, and good work comes out. That single relationship — **output quality equals context quality** — is the whole idea. Everything else is a consequence of it.

So how does context go wrong? Three ways. I call them the three C's: **Complete, Correct, Concise.**

![The same window, three ways it goes bad: a wrong fact, a missing piece, or noise crowding the signal.](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/context-engineering/images/three-failures.png)

One window, three failure modes. A piece it needed but never got. A wrong fact sitting there quietly. Or noise: full to the brim, signal buried. Every agent failure I've debugged is one of these three.

And they aren't equally bad. A wrong fact or a missing piece will break your output; noise only degrades it. So when something goes sideways, check for wrong and missing context first. That's the triage rule: **wrong, then missing, then noisy.**

Each failure has a **brake** — a deliberate move that repairs that one dimension.

![A brake for each failure: Complete fights eagerness, Correct fights misalignment, Concise fights context-rot.](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/context-engineering/images/cheatsheet.png)

Three failures, three brakes. The rest of the post zooms into each column.

---

## Act I — Complete

*The window is missing a piece it needed, and the agent acted anyway.*

The agent's worst instinct is that it acts before it knows. It defaults to doing — it races past the gathering step and writes code against a schema it never read. And missing context is usually a failure you cause yourself, by letting it run before it has investigated. That's on us, not the model.

One nuance matters here. Complete doesn't mean cram everything in; it means enough for this task. Fill the gap to good-enough, then stop. Over-gathering is a different failure — that's Act III — so don't solve a missing-context problem by creating a noise problem. It takes practice to feel the line.

**The brake: investigate before you write.** Put the agent in a research posture and name what it has to understand before it touches anything. You're turning "go build it" into "go understand it first."

Here's where I did this. I wanted to turn our [SLACspeak glossary](https://www.youtube.com/watch?v=5E3PcXXwAiU) into a searchable tool. My first messages weren't "write a scraper" — they were `/no-eager`: just visit the page, browse a few entries, and tell me you understand the layout. That holds the agent in discussion before a single edit reaches the repo. Boring on purpose.

The other half of being complete is writing down what you don't know. A good context names its own open threads:

- The keyword lists aren't final — run the ETL once, see what falls out, then extend them.
- Cross-reference edge cases ("see also X and Y") are still undecided.
- Blank definitions: keep or drop?

The agent wrote these down itself. Surfacing the unresolved questions is part of being complete — a named gap can't ambush you later.

---

## Act II — Correct

*Every piece is present, but one of them is wrong — and a wrong fact poisons everything built on top of it.*

Wrong beats missing, and badly. A missing piece announces itself: the agent gets stuck and you notice. A wrong fact looks exactly like a right one. It hallucinated REST when the API was gRPC, and every step after that inherited the lie. Silent, confident, and wrong is the worst combination there is.

Why obsess over correctness? Cost.

![Errors high in the stack multiply: one wrong line of code costs a line; one wrong line in the plan costs a hundred; one wrong assumption in research costs thousands.](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/context-engineering/images/cost-asymmetry.png)

A wrong line of code costs you a line. A wrong line in the plan costs you a hundred. A wrong assumption in research costs thousands. Errors high in the stack multiply, which is why a few minutes of alignment is the best trade you'll make: you're buying down your most expensive errors.

**The brake: agree before you act.** Clarify the ambiguity, settle scope and approach, and — the part that matters — make the agent restate the task back to you before any tool fires. If its summary is wrong, you just caught a thousand-line mistake for free.

On the SLACspeak build the same gate opened every stage: load context, clarify the gaps, get approval, then go. The stages weren't identical; the point is that a clarify-and-approval gate fronted every one of them.

Alignment also guards against beliefs you haven't checked. The agent assumed Python's `sqlite3` ships with FTS5 — usually true — but instead of betting the build on it, it flagged the assumption and wrote down the one-line command to verify on the SLAC environment. That's a correct context: it names what it's unsure of and tells you exactly how to check.

---

## Act III — Concise

*Every fact is right. Nothing is missing. And the window is still too full to think clearly.*

This is the failure you earn by succeeding for a while. Grep logs, dead ends, an eight-thousand-token file dump, retry after retry — it all piles up. The plan is still in there, just crowded out. Noise degrades the output even when every fact is correct and present.

The counterintuitive part: more context can mean worse output. There's a healthy middle band of utilization; past it, every extra blob makes the model a little dumber. More isn't better — it's a liability you have to manage.

**The brake: defend the window.** Intentional compaction, handoff docs, fresh sessions, and subagents that absorb the messy search so your main window never sees it.

What does a compaction look like? Seven parts:

![The shape of a compaction: goal, current state, decisions with why, what was ruled out, load-bearing assumptions, open threads, and pointers.](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/context-engineering/images/compaction-shape.png)

Goal, current state, decisions with their why, what you ruled out, load-bearing assumptions, open threads, pointers. The teaching isn't any one line — it's the compression: a whole build session, a full window's worth of work, distilled onto a single page a fresh agent can read cold.

And here's the hinge. The doc you wrote to compact a dying session is the same doc that lets the work continue past it. One artifact, two jobs: clean up this window, and seed the next one. That second job is the bridge into the most interesting part.

---

## The glimpse — how this scales

Front-load these brakes far enough, and one clean context becomes many, along two axes.

![Scale across two axes: across time, one clean context seeds the next; across people, the same context becomes a shared team substrate.](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/context-engineering/images/scale-time-people.png)

**Across time.** This is what I did with SLACspeak: three stages — scrape, ETL, SQLite — each a fresh agent that opens by reading the previous handoff. No single session that slowly rotted, just clean contexts chained together, each starting sharp because the last one packed its bags. Push the idea to its limit and the agent resets every iteration, with an append-only notebook as the only memory that survives. Alignment moves to design time: front-load the brakes once, and the loop runs itself.

**Across people.** Same artifacts, new axis. Several people and several agents, all reading and writing the same research-and-plan substrate on disk. The documents that compacted your solo session are the ones that keep a team aligned. You already built the team tool without knowing it.

And it's not just me. A team called HumanLayer ran this: a new intern went from two PRs on day one to ten by day eight, because the shared substrate got them productive in days, not weeks. The real win wasn't throughput, though — it was that review became keeping everyone on the same page rather than policing every line.

It's worth being straight about where this stops. It widens only as far as the pieces stay independent and each is cheap to verify. Hit a deep dependency tangle and you still need a domain expert in the loop. That's the boundary, and naming it is what makes the rest credible.

---

## To bring it home

One lever: context. Three C's:

- **Complete** — don't act on a gap.
- **Correct** — don't build on a false fact.
- **Concise** — don't crowd the signal.

And when in doubt: wrong and missing first, noise last.

If you take one thing, take this: your scarce attention belongs upstream, on intent and research and the plan, not on reviewing every line of the diff. That's the shift. You're not a typist with a fancy assistant; you're a context engineer.

The solo workflow (a context plus a handoff) and the team substrate (many agents plus a shared base) are the same shape. That's the gift: master the craft alone, and you've already learned how to do it as a team.

Context is the only thing you control, so control it on purpose.
