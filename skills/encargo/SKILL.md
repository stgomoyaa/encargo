---
name: encargo
license: MIT
description: Use when a raw prompt needs rewriting or sharpening, when the user says "rewrite this prompt", "make this prompt better", or asks for a prompt to hand to another agent. Also use before launching any autonomous, long-running, parallel or expensive run (/loop, subagent fan-out, worktrees, overnight work), and whenever a prompt states its target as taste ("until it's perfect", "make it look good", "AAA quality", "until I'm happy") or grants parallelism with no number attached.
---

# encargo

*Encargo* (Spanish): the thing you hand someone to do. Not a wish, not a topic. An assignment complete enough that they can finish it without you in the room.

A raw prompt is almost always 100% task and 0% of the five other things an agent needs to finish well. This skill turns it into an encargo: the task, plus the resources, the autonomy, the goal, the guardrails, and where the output lands.

The six-part anatomy is the easy half. What actually saves money are the **four leak checks**, derived from two real prompts whose finish lines were taste. One ran 286 commits on a browser FPS and left its author unhappy. The other asked for a shooter matching the latest Call of Duty, looped until perfect, and produced [Claude of Duty](https://github.com/mshumer/Claude-of-Duty): 55k lines, and a README admitting it does not match. Good output, and the loop still never closed — a human stopped it. An unfalsifiable goal does not ruin the work. It hands the stop decision back to you at whatever moment your patience runs out.

---

## When NOT to use this

Do not inflate small work. A one-line fix, a question, a rename, "run the tests" do not need six sections; the encargo would be longer than the job. Reach for this when the work is **autonomous, long, parallel or expensive**, or when an already-written prompt fails one of the four checks.

If the work has to survive several sessions with retries and waits, a durable-goal skill runs the loop and this skill writes the assignment that goes inside it. Hand it over explicitly; do not assume it happens. Either way the encargo still names **a file on disk** holding the ceiling, the goal and the do-not-cross. A run that gets compacted or resumed re-reads that file, not the conversation, which by then may not exist.

## The output contract

The output IS, in this order, and nothing else:

1. **A code block with the rewritten encargo**, ready to paste, holding the six sections below in that order.
2. **One line per leak check**: what it found and what it did. Four lines, not a report.
3. **One line of cost**: iterations, agents or tokens, and what happens when the ceiling is hit.

No preamble, no explaining the anatomy, no restating the original prompt.

When the work is too small to deserve an encargo, the output is one line saying so plus the direct instruction. Not a shrunken six sections.

### The six sections

| # | Section | What goes in | If it is missing |
|---|---|---|---|
| 1 | **Task** | What has to happen, in one sentence, phrased as an order. "Security audit of the API" is a subject line; "find and document every X" is an assignment. | There is no assignment. |
| 2 | **Resources** | Files, commands, branches, tools, which ports. Exact paths and names, not "the repo". With more than one agent, **what each one owns, split by a disjoint unit — path, directory, glob — never by theme**. "Auth" and "API routes" sound like separate slices and are the same files. | The agent rediscovers everything from zero, and two agents pay to redo each other's work. |
| 3 | **Autonomy** | What it may decide alone **and what stops it**. Every grant ships with its cut-off. Separate what is reversible enough to just do from what is irreversible or externally visible and stops for a human whatever the budget says. Define "report": how long, what shape. | "Keep going until it's done" with no cut-off is an infinite loop on your card. |
| 4 | **Goal** | The finish line, measurable by something other than the agent's own word. | See check 1. This is the expensive leak. |
| 5 | **Guardrails** | The quality bar **and** the named do-not-cross. If a step cannot be undone, say so, and say what undoing the rest looks like. | The agent patches downstream and touches what it should not. |
| 6 | **Output** | Where the result lands and in what format, on disk. | The work dies with the session. |

## The four leak checks

Each one is answered explicitly in the output. They are not optional.

**1 · Can the agent close the finish line, or does it need the user's taste?**

If the terminal criterion is "fun", "wants to play another round", "looks AAA", "utterly perfect", the agent cannot evaluate it, so it will either lie or iterate forever. Both cost money.

Split it three ways:

1. What the agent **can** close: a command with expected output, a count, a comparison against a named artifact. A scored rubric counts here only when its levels and its reference point were fixed **before** the run; a rubric invented mid-run and pointed at the run's own output is taste with a number stapled on.
2. An **explicit human checkpoint**: exactly what to look at, in what order.
3. When the target is genuinely out of reach, **ship plus a written gap**. "Match a AAA game" is not closeable by anyone. "Ship it, and write down exactly which parts fall short and by how much" is closeable, and it produces something better than an infinite loop: a deliverable plus an honest assessment. Each line of that gap carries the measurement behind it; a shortfall nobody measured is itself a gap to record, not a claim to make.

Option 3 is the one people skip, and the one that turns an aspirational goal into a finishable one. Otherwise the goal outsources the stop decision back to you, and you stop wherever patience runs out. The work may still be good — you just paid to find out where you would stop.

Taste is never the loop's exit condition.

**2 · Is there a ceiling?**

Autonomy without a number is spend without a number. Section 3 carries the number and the behaviour on contact; this check is only whether one exists at all. A self-paced loop over an open backlog has no ceiling.

Two phrasings imitate a ceiling. "Keep going until the request is fully resolved" is a persistence instruction sized for one turn; dropped into a loop it removes the stop condition instead of setting one.

The second is subtler. A ceiling governs work, not turns. An agent whose last message promises the next action has announced, not stopped. For unattended runs, say it outright: stopping means reporting.

**3 · Is it a task, or a roadmap in disguise?**

If the encargo contains phase 2, 3, 4 and 5, it is not a task, it is a plan, and the agent will pick its own order across a huge space with nobody seeing anything until the end. One phase per encargo. The rest is listed as "do not touch this yet" context.

**4 · Does the verifier run on this machine?**

A named verifier the environment cannot execute is worse than none: the agent silently falls back to the very thing the prompt forbade. Confirm the command exists and runs here before handing the encargo over. Real cases: "verify in a real browser" when pointer lock does not work under automation; heavy production builds on a 16GB laptop with every MCP server running. If it cannot run, swap it for one that can, or make it a human checkpoint under check 1.

## Telegraphic mode (default when the encargo is re-read many times)

An encargo that feeds a 12-iteration loop, or 8 parallel subagents, is paid for 12 or 8 times. There, prose is money — and worse than money: every low-signal word competes for the same attention the agent needs to hold the ceiling and the do-not-cross, so a padded assignment is also followed less well. In that case emit it **telegraphic**: imperatives, no articles, no copulas, no connective courtesy, one fact per line.

```
TASK  Raise visual bar. 5 surfaces: viewmodel, arena light, materials, impacts, HUD.
RESOURCES  ~/dev/iagame, branch feat/visual-pass. Refs in docs/ref/. 3 agents:
           A=viewmodel+impacts, B=arena light+materials, C=HUD. No two on one file.
AUTONOMY  You pick technique and order. Ceiling 12 iterations or 3h. On contact: STOP,
          report <=10 lines: surface, measured number, what is left. Stop means stopped,
          not announced. No push/merge without a human.
GOAL  Closes: 5 surfaces no regression, 2.5ms CPU+GPU with 10 bots prod build, tests green.
      Does NOT close: whether it looks better. Emit docs/visual-pass/before-after.png and STOP.
GUARDRAILS  Zero per-frame allocs. Do not touch src/game/net/**. Fix at source, never
            downstream. Migrations irreversible: none here, revert = git revert on branch.
OUTPUT  docs/visual-pass/before-after.png + progress.md with the measured number per surface.
        progress.md also carries ceiling, goal and do-not-cross, re-read each iteration.
```

**Never compressed:** numbers, paths, commands, verbatim strings, the ceiling, the do-not-cross, who owns which slice, and the split between what the agent closes and what the human closes. Compress the words, never the facts.

**Do not use telegraphic mode** when a human reads the encargo rather than an agent, or when precision needs a subordinate clause. A short ambiguous encargo costs more than a long clear one: the saving is tokens per re-read, not thinking less.

### Relationship to `caveman`

[caveman](https://github.com/JuliusBrussee/caveman) (MIT) is the mirror image, and the two compose rather than compete. It compresses what the agent **says**, stripping padding while leaving error text, commands and code untouched. Telegraphic mode compresses what the agent **is told**, a different token pool: an encargo is paid once per re-read, so a 12-iteration loop pays for it 12 times regardless of how terse the replies are.

Run both on long work, caveman outbound and telegraphic inbound. They are safe together for one concrete reason: caveman leaves code blocks alone, and an encargo is emitted inside a code block.

## Composing with other skills

`encargo` is a **process skill**: it defines how work is handed over, before any implementation skill does it.

| Situation | Order |
|---|---|
| The idea is still vague ("I want to build something like X") | Brainstorming first, then `encargo`. Brainstorming settles WHAT; encargo settles how it is handed over. |
| A spec or clear requirements already exist | `encargo` directly, then whichever implementation skill applies. |
| The work must survive sessions, with retries and waits | `encargo` writes it, a durable-goal skill runs the loop. Encargo hands it the goal, the verifier and the state file. |
| The encargo opens a fan-out of agents | `encargo` fixes the count, the ceiling and who owns what; the parallel-dispatch skill executes. |
| A standing code discipline is installed, such as [ponytail](https://github.com/DietrichGebert/ponytail)'s build-less ladder | Name it in Guardrails like any other project rule. Encargo bounds the assignment; that skill bounds what gets built. |
| Responses are the expensive half, not the assignment | That is `caveman`'s job, not this one. See the section above: they compress opposite directions. |
| There is a bug in the way | Systematic debugging first. An encargo aimed at a misdiagnosed cause is spend pointed at the wrong place. |

It replaces none of them. It hands them work that is already bounded, with a ceiling and a finish line.

## Standing rules

Project-wide rules live in your instructions file, not here. This skill **pulls down only the ones the work touches, with the concrete path or command**. A generic rule pasted into an encargo is noise; `commit-tool "<msg>" <files>` is a guardrail.

Most often forgotten once work is autonomous: the serialization gate before push, merge or deploy; backgrounding anything that waits, since foreground calls die on a timeout; and not spinning up heavy servers or browsers on a memory-constrained machine.

## Common mistakes

| Mistake | Fix |
|---|---|
| Rewriting the prompt longer but just as vague | Length is not specificity. If you did not add a number, a path or a command, you added nothing. |
| Inventing the success criterion | If you do not know what counts as done, that is **the** question worth asking, and it is one question. Where a baseline can be measured now, measure it instead of asking. |
| Spelling out every step instead of the finish line | That is a procedure, not a goal. Say what closes it and let the agent pick the route. |
| Leaving taste as the exit condition | Check 1. Always. |
| Handing N agents a shared surface | A count is not a partition. Name what each owns, or pay twice for the same file. |
| Turning a 3-line fix into a six-section assignment | See "When NOT to use this". |
| Duplicating durable-goal discipline | Delegate and point at it. |

## Provenance

The six-section anatomy comes from the "One Prompt, Dissected" infographic, itself a dissection of a QA prompt by Peter Steinberger. The four leak checks, the output contract and telegraphic mode are original, derived from a real `/loop` prompt and its measured result. Telegraphic mode is named to avoid colliding with [caveman](https://github.com/JuliusBrussee/caveman), which compresses agent responses rather than assignments; see the section on how they compose.

Later revisions took mechanisms, never text, from published guidance, and kept only what a baseline run of this skill actually got wrong: sub-agent scoping and the context-budget argument from Anthropic's context-engineering writing; grading criteria fixed before the run from its evaluation docs; the reversibility axis from its prompting guidance. The persistence phrasing named in check 2 as a false ceiling is OpenAI's, cited there as a trap rather than a recommendation. Advice that the baseline already satisfied was dropped rather than added.
