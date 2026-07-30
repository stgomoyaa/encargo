---
name: encargo
description: Use when a raw prompt needs rewriting or sharpening, when the user says "rewrite this prompt", "make this prompt better", or asks for a prompt to hand to another agent. Also use before launching any autonomous, long-running, parallel or expensive run (/loop, subagent fan-out, worktrees, overnight work), and whenever a prompt states its target as taste ("until it's perfect", "make it look good", "AAA quality", "until I'm happy") or grants parallelism with no number attached.
---

# encargo

*Encargo* (Spanish): the thing you hand someone to do. Not a wish, not a topic. An assignment complete enough that they can finish it without you in the room.

A raw prompt is almost always 100% task and 0% of the five other things an agent needs to finish well. This skill turns it into an encargo: the task, plus the resources, the autonomy, the goal, the guardrails, and where the output lands.

The six-part anatomy is the easy half. What actually saves money are the **four leak checks**, derived from two real prompts whose finish lines were taste. One ran 286 commits on a browser FPS and left its author unhappy. The other asked for "a first-person shooter at the level of the most recent Call of Duty, /loop until utterly perfect" and produced [Claude of Duty](https://github.com/mshumer/Claude-of-Duty), 55k lines and 2.2k stars, whose own README states the goal was to match a modern Call of Duty and that it does not. Good output, and the loop still never closed: a human stopped it. An unfalsifiable goal does not ruin the work, it hands the stop decision back to you at whatever moment your patience runs out.

---

## When NOT to use this

Do not inflate small work. A one-line fix, a question, a rename, "run the tests" do not need six sections; the encargo would be longer than the job. Reach for this when the work is **autonomous, long, parallel or expensive**, or when an already-written prompt fails one of the four checks.

If the work has to survive several sessions with retries and waits, this is not enough on its own: use a durable-goal skill for the loop, and this skill only to write the assignment that goes inside it. Do not duplicate goal discipline here.

## The output contract

The output IS, in this order, and nothing else:

1. **A code block with the rewritten encargo**, ready to paste, holding the six sections below in that order.
2. **One line per leak check**: what it found and what it did. Four lines, not a report.
3. **One line of cost**: iterations, agents or tokens, and what happens when the ceiling is hit.

No preamble, no explaining the anatomy, no restating the original prompt.

### The six sections

| # | Section | What goes in | If it is missing |
|---|---|---|---|
| 1 | **Task** | What has to happen, in one sentence. | There is no assignment. |
| 2 | **Resources** | Files, commands, branches, tools, how many agents, which ports. Exact paths and names, not "the repo". | The agent rediscovers everything from zero, and you pay for the rediscovery. |
| 3 | **Autonomy** | What it may decide alone **and what stops it**. Every grant ships with its cut-off. | "Keep going until it's done" with no cut-off is an infinite loop on your card. |
| 4 | **Goal** | The finish line, measurable by something other than the agent's own word. | See check 1. This is the expensive leak. |
| 5 | **Guardrails** | The quality bar **and** the named do-not-cross. | The agent patches downstream and touches what it should not. |
| 6 | **Output** | Where the result lands and in what format, on disk. | The work dies with the session. |

## The four leak checks

Each one is answered explicitly in the output. They are not optional.

**1 · Can the agent close the finish line, or does it need the user's taste?**

If the terminal criterion is "fun", "wants to play another round", "looks AAA", "utterly perfect", the agent cannot evaluate it, so it will either lie or iterate forever. Both cost money.

Split it three ways:

1. What the agent **can** close: a command with expected output, a count, a comparison against a named artifact.
2. An **explicit human checkpoint**: exactly what to look at, in what order.
3. When the target is genuinely out of reach, **ship plus a written gap**. "Match a AAA game" is not closeable by anyone. "Ship it, and write down exactly which parts fall short and by how much" is closeable, and it produces something better than an infinite loop: a deliverable plus an honest assessment.

Option 3 is the one people skip, and it is the one that converts an aspirational goal into a finishable one. The alternative is that the goal quietly outsources the stop decision back to the human, who ends up stopping wherever patience or budget runs out. The work may still be good. You just paid to find out where you would stop.

Taste is never the loop's exit condition.

**2 · Is there a ceiling?**

Autonomy without a number is spend without a number. Every Autonomy section declares one: N iterations, N agents, N tokens, or wall-clock time. And it declares what happens on contact: **stop and report**, never "keep going anyway". A self-paced loop over an open backlog has no ceiling.

**3 · Is it a task, or a roadmap in disguise?**

If the encargo contains phase 2, 3, 4 and 5, it is not a task, it is a plan, and the agent will pick its own order across a huge space with nobody seeing anything until the end. One phase per encargo. The rest is listed as "do not touch this yet" context.

**4 · Does the verifier run on this machine?**

A named verifier the environment cannot execute is worse than none: the agent silently falls back to the very thing the prompt forbade. Before handing over the encargo, confirm the command exists and runs here. Cases that already happened: "verify in a real browser" when pointer lock does not work in automation browsers, and heavy production builds on a 16GB laptop with every MCP server running. If the verifier cannot run, swap it for one that can, or convert it into a human checkpoint under check 1.

## Telegraphic mode (default when the encargo is re-read many times)

An encargo that feeds a 12-iteration loop, or 8 parallel subagents, is paid for 12 or 8 times. There, prose is money. In that case emit it **telegraphic**: imperatives, no articles, no copulas, no connective courtesy, one fact per line.

```
TASK  Raise visual bar. 5 surfaces: viewmodel, arena light, materials, impacts, HUD.
RESOURCES  ~/dev/iagame, branch feat/visual-pass. Refs in docs/ref/. 3 agents.
AUTONOMY  You pick technique and order. Ceiling 12 iterations or 3h. On contact: STOP, report.
GOAL  Closes: 5 surfaces no regression, 2.5ms CPU+GPU with 10 bots prod build, tests green.
      Does NOT close: whether it looks better. Emit docs/visual-pass/before-after.png and STOP.
GUARDRAILS  Zero per-frame allocs. Do not touch src/game/net/**. Fix at source, never downstream.
OUTPUT  docs/visual-pass/before-after.png + progress.md with the measured number per surface.
```

**Never compressed:** numbers, paths, commands, verbatim strings, the ceiling, the do-not-cross, and the split between what the agent closes and what the human closes. Compress the words, never the facts.

**Do not use telegraphic mode** when a human reads the encargo rather than an agent, or when precision needs a subordinate clause. A short ambiguous encargo costs more than a long clear one: the saving is tokens per re-read, not thinking less.

### Relationship to `caveman`

[caveman](https://github.com/JuliusBrussee/caveman) (MIT) does the mirror image of this and the two compose rather than compete. It compresses what the agent **says**, cutting response tokens by roughly 65% by dropping conversational padding while keeping code, commands and error messages byte-for-byte. Telegraphic mode compresses what the agent **is told**, which is a different token pool: an encargo is paid once per re-read, so a 12-iteration loop pays for it 12 times whether or not the responses are compressed.

Run both when the work is long: caveman on the way out, telegraphic on the way in. They do not conflict, and there is one concrete reason they are safe together: caveman preserves code blocks verbatim, and an encargo is emitted inside a code block, so an encargo survives caveman intact.

## Composing with other skills

`encargo` is a **process skill**: it defines how work is handed over, before any implementation skill does it.

| Situation | Order |
|---|---|
| The idea is still vague ("I want to build something like X") | Brainstorming first, then `encargo`. Brainstorming settles WHAT; encargo settles how it is handed over. |
| A spec or clear requirements already exist | `encargo` directly, then whichever implementation skill applies. |
| The work must survive sessions, with retries and waits | `encargo` writes it, a durable-goal skill runs the loop. Encargo hands it the goal and the verifier. |
| The encargo opens a fan-out of agents | `encargo` fixes the count and the ceiling; the parallel-dispatch skill executes. |
| Responses are the expensive half, not the assignment | That is `caveman`'s job, not this one. See the section above: they compress opposite directions. |
| There is a bug in the way | Systematic debugging first. An encargo aimed at a misdiagnosed cause is spend pointed at the wrong place. |

It replaces none of them. It hands them work that is already bounded, with a ceiling and a finish line.

## Standing rules

Project-wide rules live in your instructions file, not here. What this skill does is **pull them down into Guardrails with the concrete path or command, and only the ones the work touches**. A generic rule pasted into an encargo is noise; `commit-tool "<msg>" <files>` or "run the design linter before closing" is a guardrail.

The ones most often forgotten once work is autonomous: the serialization gate before push, merge to the default branch, or deploy; backgrounding anything that waits, because foreground tool calls get killed on a timeout; and not spinning up heavy dev servers or browsers on a memory-constrained machine.

## Common mistakes

| Mistake | Fix |
|---|---|
| Rewriting the prompt longer but just as vague | Length is not specificity. If you did not add a number, a path or a command, you added nothing. |
| Inventing the success criterion | If you do not know what counts as done, that is **the** question worth asking, and it is one question. |
| Leaving taste as the exit condition | Check 1. Always. |
| Turning a 3-line fix into a six-section assignment | See "When NOT to use this". |
| Duplicating durable-goal discipline | Delegate and point at it. |

## Provenance

The six-section anatomy comes from the "One Prompt, Dissected" infographic, itself a dissection of a QA prompt by Peter Steinberger. The four leak checks, the output contract and telegraphic mode are original, derived from a real `/loop` prompt and its measured result. Telegraphic mode is named to avoid colliding with [caveman](https://github.com/JuliusBrussee/caveman), which compresses agent responses rather than assignments; see the section on how they compose.
