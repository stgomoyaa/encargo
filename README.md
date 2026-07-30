<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/stgomoyaa/encargo/main/assets/repo-banner-encargo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/stgomoyaa/encargo/main/assets/repo-banner-encargo-light.svg">
    <img width="880" alt="encargo. Rewrite a raw prompt into an assignment an agent can actually finish. Stack: Markdown, Zero deps, 4 leak checks." src="https://raw.githubusercontent.com/stgomoyaa/encargo/main/assets/repo-banner-encargo-light.svg">
  </picture>
</p>

# encargo

**A skill that rewrites your prompt into an assignment an agent can actually finish.**

*Encargo* (Spanish): the thing you hand someone to do. Not a wish, not a topic. Complete enough that they can finish it without you in the room.

Most prompts are 100% task and 0% of the five other things an agent needs: what resources it has, how much it may decide alone, what counts as done, what it must not touch, and where the output goes. `encargo` fills those in, and then runs four checks that the anatomy alone does not catch.

## The problem this fixes

Two real prompts of mine, both instructive.

The first ran a browser FPS for **286 commits** and left me unhappy with the result. It was not a sloppy prompt. It had all six sections, a written finish line, hard-won lessons, a do-not-cross list. It still burned money, because of this line:

> The game is finished when I can open it, pick a loadout, play a deathmatch against bots, win or lose RR, **and want to play another round.**

An agent cannot evaluate "want to play another round". So it either declares victory without evidence, or it never stops. Both are expensive. Nothing in the prompt capped iterations either.

The second is the one worth studying, because it did not fail:

> It should be utterly perfect, visually beautiful, with every single thing done at AAA quality. Fan out sub-agents. /loop until it's utterly perfect. Don't stop until each sub-agent is utterly wowed compared with the actual Call of Duty. It should literally compare them side by side blind and say which one looks better.

That prompt produced [Claude of Duty](https://github.com/mshumer/Claude-of-Duty): 55k lines across 11 subsystems, every texture, mesh, animation and sound generated procedurally at load time, 2.2k stars. Genuinely impressive output. An unfalsifiable goal does not prevent good work.

What it prevents is **the loop ever closing**. Read that repo's own "Honest assessment" section: the stated goal was to match a modern Call of Duty, and it says plainly that it does not, then lists the gaps in hand animation, material realism, character quality, indirect lighting and frame rate. The exit condition, "don't stop until each sub-agent is utterly wowed", was never met. A human stopped it. And the title says "a single prompt" while the README describes three rounds of six-agent passes, which is a human steering, not a prompt terminating.

So the pattern is not "vague prompt, bad output". It is:

> **An unfalsifiable goal outsources the stop decision back to you, at whatever moment you happen to run out of patience or budget.** The work might be great. You just paid to discover where you'd stop.

Both prompts fail the same way: **the finish line is taste, and taste cannot close a loop.** Claude of Duty ends with a written gap and 2.2k stars. My FPS ended with 286 commits and no such paragraph, because I kept iterating against a target instead of shipping and naming the distance to it. Same prompt shape, two very different endings, and the difference was not the prompt.

## The four leak checks

This is the part you will not find in a prompt-anatomy diagram.

| # | Check | The fix it forces |
|---|---|---|
| 1 | Can the agent close the finish line, or does it need your taste? | Split it three ways: what a command can verify, an explicit human checkpoint, and, when the goal is genuinely out of reach, **ship plus a written gap**. "Reach AAA" is not closeable; "ship it and write down exactly how far off it is" is. Taste is never the loop's exit condition. |
| 2 | Is there a ceiling? | N iterations, N agents, N tokens or wall-clock, **and** what happens on contact: stop and report. |
| 3 | Is it a task or a roadmap in disguise? | One phase per encargo. The other phases become "do not touch yet" context. |
| 4 | Does the verifier run on this machine? | A verifier the environment cannot execute is worse than none: the agent silently falls back to what the prompt forbade. Confirm it runs, or convert it to a human checkpoint. |

## The six sections

| # | Section | What goes in |
|---|---|---|
| 1 | **Task** | What has to happen, in one sentence. |
| 2 | **Resources** | Exact paths, commands, branches, tools, agent count, ports. Not "the repo". |
| 3 | **Autonomy** | What it may decide alone, and what stops it. Every grant ships with its cut-off. |
| 4 | **Goal** | The finish line, measurable by something other than the agent's own word. |
| 5 | **Guardrails** | The quality bar and the named do-not-cross. |
| 6 | **Output** | Where the result lands, on disk, in what format. |

## Caveman mode

An encargo feeding a 12-iteration loop is paid for 12 times. An encargo fanned out to 8 subagents, 8 times. There, prose is money, so the encargo is emitted telegraphic:

```
TASK  Raise visual bar. 5 surfaces: viewmodel, arena light, materials, impacts, HUD.
RESOURCES  ~/dev/iagame, branch feat/visual-pass. Refs in docs/ref/. 3 agents.
AUTONOMY  You pick technique and order. Ceiling 12 iterations or 3h. On contact: STOP, report.
GOAL  Closes: 5 surfaces no regression, 2.5ms CPU+GPU with 10 bots prod build, tests green.
      Does NOT close: whether it looks better. Emit docs/visual-pass/before-after.png and STOP.
GUARDRAILS  Zero per-frame allocs. Do not touch src/game/net/**. Fix at source, never downstream.
OUTPUT  docs/visual-pass/before-after.png + progress.md with the measured number per surface.
```

Words get compressed. Numbers, paths, commands, verbatim strings, the ceiling, the do-not-cross, and the human/agent split never do.

## Install

The skill is one markdown file. Any harness that reads `SKILL.md` directories can use it.

```bash
git clone https://github.com/stgomoyaa/encargo.git
cp -R encargo/skills/encargo ~/.claude/skills/
```

Project-local works too, in `<project>/.claude/skills/encargo/`. Other harnesses: same folder, different destination. `~/.agents/skills/` for the `.agents` convention, `.cursor/skills/` for Cursor with Agent Skills enabled. Only the Claude Code path is verified here.

## Make it automatic

The skill triggers on its own description, but the highest-value moment is the one you will forget: right before launching something autonomous. Add a line to your `CLAUDE.md` / `AGENTS.md`:

```markdown
Before launching any autonomous, long-running, parallel or expensive work
(/loop, subagent fan-out, worktrees, overnight runs), invoke the `encargo`
skill first. The four leak checks are not optional.
```

## What it is not

Not a prompt library, not a template you fill in, not a wrapper that makes prompts longer. It is a contract with four checks, and it explicitly refuses to inflate small work: a one-line fix does not get six sections.

It also does not do goal loops. If work must survive several sessions with retries and waits, `encargo` writes the assignment and hands the goal and verifier to a durable-goal skill. Composition, not duplication.

## Provenance

The six-section anatomy comes from the ["One Prompt, Dissected"](https://x.com/steipete) infographic, itself a dissection of a QA prompt by Peter Steinberger. The four leak checks, the output contract and caveman mode are original, derived from the two prompts above and their measured results.

MIT licensed. Built by [Santiago Moya](https://github.com/stgomoyaa).
