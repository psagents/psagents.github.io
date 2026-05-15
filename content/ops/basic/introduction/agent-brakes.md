# Agent brakes: working with a skilled but unpredictable partner

Here's the default experience of working with an agent. You type a task, hit enter, and the agent is gone — reading files, writing code, running commands. For pure retrieval — reading, searching, explaining — that's fine; nothing is at stake, so let it run. But the moment a task *changes state* — your code, your files, your configs — the calculus flips. A minute later you're handed a finished result that might even work — but if you and the agent weren't aligned on what you wanted, you're now picking through it to find where it drifted. The review you do at the end is only as cheap as the alignment you did up front.

The problem isn't capability. It's that the agent's default is to *act* — and whether it pauses to check with you first is unpredictable. **Agent brakes** are a small set of slash-command skills that give you an explicit lever for that pause: they let the agent absorb context before it commits to an approach, check that it understood you, and only release it when you're ready.

## Three axes

![The three brakes cheatsheet](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/00-agent-brakes.png)
*The cheatsheet: three axes — eagerness, alignment, context-rot — and the brake skill that answers each.*

Brakes restrain agent behavior along three axes: **eagerness** (let it absorb context before committing, so it doesn't lock in a premature, locally-optimal solution), **alignment** (make it surface what it thinks you meant), and **context-rot** (reset state cleanly). The cheatsheet above maps each failure to the brake that fixes it.

The important thing: these brakes are just prompts. They aren't tied to a particular tool or model. The walkthrough below runs in **OpenCode on Claude Sonnet 4.6, via the SLAC AI Gateway** — not Claude Code. Same brakes, different harness. That portability is the whole point.

## The real idea: forming the task

It's tempting to think of brakes as friction on a task you already have. But the task usually *isn't* clear when you start. `no-eager`, `clarify`, and `align` aren't speed bumps — they're how you develop the idea of the task, just in time. Only once it's solid do you reach for `approval` to commit, and `go` to release.

So watch a vague intent — "read an image with psana2" — become a concrete, executable task, without you ever losing the thread.

## The walkthrough

The task is deliberately small: use PSANA2 (an internal LCLS data tool) to read a detector image. Small enough that you can hold the whole thing in your head — so what stands out is the *mechanism*, not the task.

### no-eager — research, don't act

![Typing the task with /no-eager](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/02-no-eager.png)
*The session is running in OpenCode; I type `/no-eager` followed by the task.*

The session is running in OpenCode. I lead with `/no-eager` and the task, pointing it at the `ask-lcls2` docs skill. The brake expands into a posture: respond in conversation only, no tool calls that mutate anything.

![The no-eager prompt expands](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/03-prompt-expand.png)
*The `/no-eager` skill expands into a full posture before the agent does anything.*
![Research done](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/04-research-done.png)
*The agent's research: the psana setup, the DataSource pattern, the image methods.*

The agent researches and reports back — the psana `DataSource` pattern, the three image methods (`raw`, `calib`, `image`), the environment setup. No code written, nothing to review yet. Just shared understanding.

### clarify — surface the gaps

![/clarify](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/05-clarify.png)
*`/clarify` makes the agent ask before it assumes.*
![Answering the questions inline](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/06-clarify-I-answer-questions.png)
*The agent's questions — experiment, run, detector, calibrated or raw — answered inline.*
![Typing /no-eager mid-clarify](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/07-no-eager.png)
*One open question — the detector name — is worth researching on its own, so I drop into `/no-eager` for it.*
![The no-eager research lands](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/07-no-eager-done.png)
*The agent explains the detector-discovery options and stops — no action taken.*

`/clarify` makes the agent surface the gaps *it* sees: which experiment? which run? which detector? calibrated or raw? I answer inline — "pick one", "single run", "help me discover the detector name".

Brakes aren't a fixed sequence, and they nest. One of those open questions — the detector name — was itself worth researching, so mid-clarify I reached for `/no-eager` to dig into just that, then came back and kept clarifying. Reach for whichever brake the moment calls for, as many times as it calls for it.

![/clarify again](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/08-clarify.png)
*Second pass: `/clarify` once more, now that the first answers opened new questions.*
![Clarify wraps up — no remaining gaps](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/09-clarify-done.png)
*The agent confirms it has no remaining questions.*

### align — check that it got your intent

![/align](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/10-align.png)
*`/align` asks the agent to restate the task back to me.*
![Align restates the task](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/11-align-done.png)
*Scope, target, method, done-when — laid out so I can catch a misread.*

`/clarify` closes unknowns. `/align` verifies the *knowns*. The agent restates the task back along four dimensions — scope, target, method, done-when — and the check runs in your head, against its words. If it misread your intent, this is where you catch it cheaply, before any code exists.

### approval — commit to the plan

![/approval](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/12-approval.png)
*`/approval` — the agent must lay out a full plan and wait.*
![The full plan, waiting for an explicit yes](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/13-approval-done.png)
*The plan, top to bottom, stopped on an explicit yes from me.*

`/approval` makes the agent write the full plan top-to-bottom and then *stop*. No ambiguous "looks good, proceeding" — it waits for an explicit yes. This is the moment the task becomes committed.

### go — release the brakes

![go](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/14-go.png)
*`go` — brakes off.*
![The script runs](https://raw.githubusercontent.com/carbonscott/.discussion/main/posts/agent-brakes/figures/15-go-done.png)
*The script is written and run; the output appears, the `psana` import warning and all.*

`go`. Brakes off. The script gets written and run, the output appears. Even the LSP error about the `psana` import is expected — and the agent explains why. The review at this point is fast: there's nothing in the result I hadn't already aligned on.

## What's left: context-rot

This task was small enough that only two axes fired — eagerness and alignment. The third, **context-rot**, shows up when a task runs long enough to span both the forming and the doing: stale assumptions pile up, the agent drifts, the window fills with dead ends. The brakes for that — `/no-op` to absorb context without acting, `/handoff` to reset cleanly into a fresh agent — deserve their own post.

But the move is the same at every scale. The agent is fast — the brakes just make sure it's fast at the *right* thing.

---

The brakes themselves — the slash-command skills — live at [github.com/carbonscott/agent-brakes](https://github.com/carbonscott/agent-brakes). You might not want to install them all; since they're just prompts, you can copy and paste them as needed.
