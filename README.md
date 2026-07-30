<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/stgomoyaa/encargo/main/assets/repo-banner-encargo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/stgomoyaa/encargo/main/assets/repo-banner-encargo-light.svg">
    <img width="880" alt="encargo. Rewrite a raw prompt into an assignment an agent can actually finish. Stack: Markdown, Zero deps, 4 leak checks." src="https://raw.githubusercontent.com/stgomoyaa/encargo/main/assets/repo-banner-encargo-light.svg">
  </picture>
</p>

# encargo

*The thing you hand someone to do. Not a wish, not a topic.*

**One raw prompt in, one assignment out: 6 sections, 4 leak checks, a ceiling and a stop condition.**

[Before / After](#before--after) · [Install](#install) · [Prose vs telegraphic](#prose-vs-telegraphic) · [What you get](#what-you-get) · [The four leak checks](#the-four-leak-checks) · [When this loses](#when-this-loses) · [How it works](#how-it-works) · [Provenance](#provenance-and-license)

`encargo` is a skill that turns a raw prompt into an encargo: a task, its resources, how much the agent may decide alone, a finish line something other than the agent's own word can check, the guardrails, and where the output lands. Hand it a prompt and you get back a ready to paste assignment in a code block, one line per leak check on what it found and fixed, and one line on cost, how many iterations, agents or tokens before the run has to stop and report.

## Before / After

The raw prompt below is real. It is the one that produced [Claude of Duty](https://github.com/mshumer/Claude-of-Duty): 55k lines across 11 subsystems, 2.2k stars, and its own "Honest assessment" section, which states "the goal was to match a modern Call of Duty" and then, in bold, "It does not."

> It should be utterly perfect, visually beautiful, with every single thing done at AAA quality. Fan out sub-agents. /loop until it's utterly perfect. Don't stop until each sub-agent is utterly wowed compared with the actual Call of Duty. It should literally compare them side by side blind and say which one looks better.

Same request, rewritten:

```
TASK  Build a browser FPS in Three.js: movement, shooting, one map, bot AI, all assets procedural.
RESOURCES  New repo. Three.js. 6 subagents.
AUTONOMY  Agents pick technique and asset pipeline. Ceiling: 8 iterations or 6 hours. On contact: STOP, report.
GOAL  Closes: playable build, deathmatch vs bots holds 60fps, tests green.
      Does NOT close: whether it looks AAA, nobody on this machine can judge that against a shipped game. Ship it, then write GAP.md: name every place it falls short of a modern Call of Duty and by how much.
GUARDRAILS  No external art or audio assets, procedural or generated only.
OUTPUT  /build, plus GAP.md with the named shortfalls, not a score.
```

| | Words |
|---|---|
| Raw prompt (the whole ask, as written) | 53 |
| Rewritten encargo (task, resources, autonomy, goal, guardrails, output) | 111 |

The rewrite is more than double the length. That is not this tool being verbose, it is the five sections the raw prompt never had, made explicit. That is the honest cost, and it runs the other way from what you might expect.

## What it doesn't do

`encargo` does not make the work better. It does not touch code, design or quality, and it cannot rescue a misdiagnosed problem. What it does is make the assignment finishable, and it explicitly refuses to inflate small work.

| | |
|---|---|
| Makes the work better | No. |
| Rescues a misdiagnosed problem | No. |
| Inflates a one-line fix into six sections | No, on purpose. It refuses. |
| Makes the assignment finishable | Yes. That is the whole job. |
| Prompt length on autonomous work | Grows. 53 words became 111 above. That growth is the six sections doing their job, not overhead to apologize for. |
| Prompt length on small work | Unchanged. Check 3 catches it before a single section gets added. |

> Not a prompt library, not a template, not a wrapper that pads prompts to look thorough. A contract with four checks, and the four checks are what decide whether the prompt needed expanding at all.

## Install

The skill is one markdown file. Any harness that reads `SKILL.md` folders under a skills directory can use it.

```bash
git clone https://github.com/stgomoyaa/encargo.git && cp -R encargo/skills/encargo ~/.claude/skills/
```

Verified on Claude Code, where it also works project local at `<project>/.claude/skills/encargo/`. The same folder should work under the `.agents/skills/` convention and Cursor with Agent Skills enabled, unverified here.

Safe to re-run: re-cloning and re-copying overwrites the same files in place. Nothing duplicates or appends.

> [!TIP]
> Add this to your `CLAUDE.md` / `AGENTS.md` so the highest value moment does not get skipped, the one right before anything autonomous starts:
> ```
> Before launching any autonomous, long-running, parallel or expensive work
> (/loop, subagent fan-out, worktrees, overnight runs), invoke the `encargo`
> skill first. The four leak checks are not optional.
> ```

## Prose vs telegraphic

Same six sections, two settings. Prose for a human reading it once, telegraphic for an assignment that gets re-read inside a loop or a fan-out, where every re-read is paid for again.

| Section | Prose | Telegraphic |
|---|---|---|
| Task | Raise the visual bar across five surfaces, the viewmodel, arena lighting, materials, hit impacts and the HUD. | Raise visual bar. 5 surfaces: viewmodel, arena light, materials, impacts, HUD. |
| Resources | Work in ~/dev/iagame on the feat/visual-pass branch. Reference images live in docs/ref/. Three agents on this. | ~/dev/iagame, branch feat/visual-pass. Refs in docs/ref/. 3 agents. |
| Autonomy | You choose the technique and the order across the five surfaces. Ceiling: 12 iterations or 3 hours, whichever comes first. On contact with either: stop and report. | You pick technique and order. Ceiling 12 iterations or 3h. On contact: STOP, report. |
| Goal | Closes when all five surfaces show no regression, the build holds 2.5ms combined CPU and GPU cost with 10 bots in a production build, and the test suite is green. Does not close on whether it looks better, that is not something the run can verify. Emit a before and after image at docs/visual-pass/before-after.png and stop. | Closes: 5 surfaces no regression, 2.5ms CPU+GPU with 10 bots prod build, tests green. Does NOT close: whether it looks better. Emit docs/visual-pass/before-after.png and STOP. |
| Guardrails | Zero per-frame allocations, none. Do not touch anything under src/game/net/. Fix at the source, never patch downstream. | Zero per-frame allocs. Do not touch src/game/net/**. Fix at source, never downstream. |
| Output | Save the before and after image to docs/visual-pass/before-after.png, and write progress.md with the measured number for each surface. | docs/visual-pass/before-after.png + progress.md with the measured number per surface. |

Same six facts, 172 words against 85, roughly half.

> [!NOTE]
> Telegraphic pays off on re-reads: a 12-iteration loop reads the encargo 12 times, an 8-way fan-out reads it 8 times. A human reading it once should get prose. A short ambiguous encargo costs more than a long clear one, the saving is tokens per re-read, not thinking less.

## What you get

The output is this, in this order, and nothing else. No preamble, no restated prompt, no explanation of the anatomy inside the output itself.

| You get | Contents |
|---|---|
| The rewritten encargo | One code block, ready to paste: Task, Resources, Autonomy, Goal, Guardrails, Output, in that order. |
| Four leak check lines | One line per check, what it found and what it did about it. |
| One cost line | Iterations, agents or tokens, and what happens on contact with the ceiling. |

## The four leak checks

This is the part you will not find in a prompt anatomy diagram.

| # | Check | The real failure it prevents |
|---|---|---|
| 1 | Can the agent close the finish line, or does it need your taste? | An unfalsifiable goal ("AAA", "fun", "utterly perfect") makes the agent either lie about being done or iterate forever. Split it three ways: what a command can verify, an explicit human checkpoint, or, when the target is genuinely out of reach, ship plus a written gap. Taste is never the exit condition. |
| 2 | Is there a ceiling? | Autonomy with no number is spend with no number. N iterations, N agents, N tokens or wall clock, and what happens on contact: stop and report, never keep going anyway. |
| 3 | Is it a task, or a roadmap in disguise? | Phase 2 through 5 hiding inside phase 1 means the agent picks its own order across a huge space, and nobody sees anything until the end. One phase per encargo. The rest becomes "do not touch yet" context. |
| 4 | Does the verifier run on this machine? | A verifier the environment cannot execute is worse than none, the agent silently falls back to whatever the prompt forbade. Confirm it runs here, or convert it to a human checkpoint. |

## When this loses

Six sections and four checks are overhead. They earn that overhead back on work that is autonomous, long, parallel or expensive, and they cost real tokens on everything else.

- A one-line fix, a rename, a question, "run the tests": none of that needs six sections. The encargo would end up longer than the job.
- Every extra section is tokens spent before the agent does anything. On a short task those tokens are pure loss, there is no re-read to earn them back against.
- It cannot rescue a misdiagnosed problem. If the bug is in the wrong file, a precisely specified encargo aimed at that file just fails more precisely. Diagnose first, write the assignment second.
- It does not run the loop itself. If the work has to survive multiple sessions with retries and waits, this writes the assignment and hands the goal and the verifier to a durable-goal skill, then stops.

> [!IMPORTANT]
> The four checks only work on a goal that is already aimed correctly. A precise, well-ceilinged assignment pointed at the wrong root cause still burns the whole ceiling, and reports failure precisely.

## How it works

1. You hand it a raw prompt.
2. It runs the goal through check 1: closeable by a command, an explicit human checkpoint, or, if genuinely out of reach, ship plus a written gap. Taste never becomes the exit condition.
3. It sets a ceiling: iterations, agents, tokens or wall clock, and what happens on contact (check 2).
4. It confirms the ask is one phase, not a roadmap wearing a task's clothes (check 3), and pushes the rest into "do not touch yet."
5. It confirms the verifier named in the goal actually runs in this environment (check 4), or swaps it for one that does.
6. It fills the six sections, Task, Resources, Autonomy, Goal, Guardrails, Output, and emits them as one code block.
7. If the encargo will be re-read many times, a loop, a fan-out, it emits telegraphic instead of prose.
8. It reports four lines, one per check, plus one line of cost. Nothing else.

## Relationship to caveman

[caveman](https://github.com/JuliusBrussee/caveman) (MIT, ~94k stars) compresses what the agent **says**: it cuts response tokens by dropping conversational padding while keeping code, commands and error messages byte for byte. Telegraphic mode compresses what the agent **is told**, a different token pool. An encargo is paid once per re-read, a 12-iteration loop pays for it 12 times whether or not the responses are compressed.

They compose rather than compete: caveman on the way out, telegraphic on the way in. One concrete reason they are safe together: caveman preserves code blocks verbatim, and an encargo is emitted inside a code block, so it survives caveman intact.

## Provenance and license

The six-section anatomy comes from the ["One Prompt, Dissected"](https://x.com/steipete) infographic, itself a dissection of a QA prompt by Peter Steinberger. The four leak checks, the output contract and telegraphic mode are original, derived from the two prompts above and their measured results. Telegraphic mode is deliberately not called "caveman mode": caveman is an existing skill that compresses agent responses, and reusing its name for something that compresses assignments would be wrong in both directions.

MIT licensed. Built by [Santiago Moya](https://github.com/stgomoyaa), who ran out of patience at commit 286 and wrote this instead.
