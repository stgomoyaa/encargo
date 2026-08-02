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

The raw prompt below is real. It is the one that produced [Claude of Duty](https://github.com/mshumer/Claude-of-Duty): 55k lines across 11 subsystems, thousands of stars, and its own "Honest assessment" section, which states "the goal was to match a modern Call of Duty" and then, in bold, "It does not."

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

### "So just write a better prompt"

Fair objection to the example above, which is sloppy. So here is a well-written one, and it fails anyway.

A real prompt asking for a single-file WebGL shooter opens with six numbered requirement sections — lighting engine, PBR materials, a post-processing stack, architecture, physics and destruction, particle systems — names its CDN dependencies, forbids external assets, and explicitly bans `// TODO` comments and placeholder markup. It is specific, technical and disciplined. It still fails three of the four checks:

| Check | What fires |
|---|---|
| 3 · roadmap in disguise | Seven subsystems wearing one task's clothes. The agent picks its own order across all of them and nobody sees anything until the end. |
| 4 · verifier runs here | It requires PointerLockControls. Pointer lock does not engage under headless automation, so nothing in the run can confirm the thing works. |
| 1 · taste as the finish line | "Hyper-realistic", "gritty", "high-detail". The one real number, 60fps, has nothing measuring it. |

There is also an unnamed ceiling: "no truncation, no placeholders" runs straight into a finite output budget, and the prompt never acknowledges the collision.

```
TASK  Phase 1 only: one .html, Three.js scene, PointerLock FPS controller, one room,
      procedural PBR materials. Phases 2-5 are listed below and do not start.
RESOURCES  Three.js r128 + PointerLockControls from CDN. No asset files, no external
           textures. NOT this phase: post-processing, physics/destruction, particles,
           viewmodel, multi-level architecture.
AUTONOMY  You pick geometry and shader technique. The real ceiling is the output budget:
          if phase 1 will not fit complete in one response, STOP and report what to cut.
          Never truncate, never emit a TODO.
GOAL  Closes: opens in Chrome with 0 console errors, pointer lock engages on click, WASD
      moves the camera, holds 60fps with the scene loaded.
      Human checkpoint, not agent-closable: pointer lock does not engage under headless
      automation, so a person opens the file and checks lock, then movement, then fps.
      Does NOT close: "hyper-realistic", "AAA", "gritty". Ship it, then write GAP.md
      naming where it falls short and by how much.
GUARDRAILS  Single file. No dependency beyond those two CDN scripts. No placeholder markup.
OUTPUT  scene.html + GAP.md. Phases 2-5 go in ROADMAP.md, unstarted.
```

521 words in, 175 out. This rewrite is a third of the length of the prompt it replaces, because specificity was never the missing ingredient — a finish line and one phase were. A detailed prompt and a finishable one are different things, and the first does not imply the second.

## What it doesn't do

`encargo` does not make the work better. It does not touch code, design or quality, and it cannot rescue a misdiagnosed problem. What it does is make the assignment finishable, and it explicitly refuses to inflate small work.

| | |
|---|---|
| Makes the work better | No. |
| Rescues a misdiagnosed problem | No. |
| Inflates a one-line fix into six sections | No, on purpose. It refuses. |
| Makes the assignment finishable | Yes. That is the whole job. |
| Prompt length on autonomous work | Goes either way. A thin prompt grows (53 words to 111); an over-scoped one shrinks (521 to 175). It moves toward one bounded phase, not toward a word count. |
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
| Resources | Work in ~/dev/iagame on the feat/visual-pass branch. Reference images live in docs/ref/. Three agents, split by file: A takes the viewmodel and impacts, B the arena lighting and materials, C the HUD. No two on one file. | ~/dev/iagame, branch feat/visual-pass. Refs in docs/ref/. 3 agents: A=viewmodel+impacts, B=arena light+materials, C=HUD. No two on one file. |
| Autonomy | You choose the technique and the order across the five surfaces. Ceiling: 12 iterations or 3 hours, whichever comes first. On contact with either: stop and report, in ten lines or fewer, giving the surface, the measured number and what is left. Stopping means stopping, not announcing. Nothing gets pushed or merged without a human. | You pick technique and order. Ceiling 12 iterations or 3h. On contact: STOP, report <=10 lines: surface, measured number, what is left. Stop means stopped, not announced. No push/merge without a human. |
| Goal | Closes when all five surfaces show no regression, the build holds 2.5ms combined CPU and GPU cost with 10 bots in a production build, and the test suite is green. Does not close on whether it looks better, that is not something the run can verify. Emit a before and after image at docs/visual-pass/before-after.png and stop. | Closes: 5 surfaces no regression, 2.5ms CPU+GPU with 10 bots prod build, tests green. Does NOT close: whether it looks better. Emit docs/visual-pass/before-after.png and STOP. |
| Guardrails | Zero per-frame allocations, none. Do not touch anything under src/game/net/. Fix at the source, never patch downstream. Nothing here is irreversible; undoing the rest is a git revert on the branch. | Zero per-frame allocs. Do not touch src/game/net/**. Fix at source, never downstream. Nothing irreversible here, revert = git revert on branch. |
| Output | Save the before and after image to docs/visual-pass/before-after.png, and write progress.md with the measured number for each surface. progress.md also carries the ceiling, the goal and the do-not-cross, to be re-read on every iteration. | docs/visual-pass/before-after.png + progress.md with the measured number per surface. progress.md also carries ceiling, goal and do-not-cross, re-read each iteration. |

Same six facts, 230 words against 125, roughly half.

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
| 1 | Can the agent close the finish line, or does it need your taste? | An unfalsifiable goal ("AAA", "fun", "utterly perfect") makes the agent either lie about being done or iterate forever. Split it three ways: what a command can verify, an explicit human checkpoint, or, when the target is genuinely out of reach, ship plus a written gap. A scored rubric only counts if its levels and reference point were fixed *before* the run — one invented mid-run and aimed at the run's own output is taste with a number stapled on. Keep a harsh critic to generate that gap list, pointed at rendered output and never the agent that built the thing; just never make "the critic went quiet" the exit, because a critic told to be harsh never does. Taste is never the exit condition. |
| 2 | Is there a ceiling? | Autonomy with no number is spend with no number. N iterations, N agents, N tokens or wall clock, and what happens on contact: stop and report, never keep going anyway. Two phrasings imitate a ceiling without being one: "keep going until the request is fully resolved" is a single-turn persistence instruction that *removes* the stop condition inside a loop, and a last message promising the next action is an announcement, not a stop. |
| 3 | Is it a task, or a roadmap in disguise? | Phase 2 through 5 hiding inside phase 1 means the agent picks its own order across a huge space, and nobody sees anything until the end. One phase per encargo. The rest becomes "do not touch yet" context. |
| 4 | Does the verifier run on this machine, and does it run first? | A verifier the environment cannot execute is worse than none, the agent silently falls back to whatever the prompt forbade. Confirm it runs here, or convert it to a human checkpoint. Then put it first as its own step: a harness written after the work gets shaped to agree with what already exists. One goal often needs two verifiers in two environments — a headless capture proves a frame looks right and proves nothing about whether it holds framerate on the machine it runs on. |

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
6. It fills the six sections, Task, Resources, Autonomy, Goal, Guardrails, Output, and emits them as one code block. On a fan-out that means naming what each agent owns, not just how many there are, and what a returned report looks like.
7. If the encargo will be re-read many times, a loop, a fan-out, it emits telegraphic instead of prose.
8. It reports four lines, one per check, plus one line of cost. Nothing else.

## Relationship to caveman

[caveman](https://github.com/JuliusBrussee/caveman) (MIT) compresses what the agent **says**: it cuts response tokens by stripping conversational padding while leaving error text, commands and code untouched. Telegraphic mode compresses what the agent **is told**, a different token pool. An encargo is paid once per re-read, a 12-iteration loop pays for it 12 times whether or not the responses are compressed.

They compose rather than compete: caveman on the way out, telegraphic on the way in. One concrete reason they are safe together: caveman leaves code blocks alone, and an encargo is emitted inside a code block, so it survives caveman intact.

[ponytail](https://github.com/DietrichGebert/ponytail) (MIT) sits on a third axis: it bounds what gets *built*. Nothing to compose there, just a place to point at — if it is installed, name its ladder in Guardrails like any other project rule.

## Provenance and license

The six-section anatomy comes from the ["One Prompt, Dissected"](https://x.com/steipete) infographic, itself a dissection of a QA prompt by Peter Steinberger. The four leak checks, the output contract and telegraphic mode are original, derived from the two prompts above and their measured results. Telegraphic mode is deliberately not called "caveman mode": caveman is an existing skill that compresses agent responses, and reusing its name for something that compresses assignments would be wrong in both directions.

A later revision read published prompting and context-engineering guidance from Anthropic and OpenAI, plus neighbouring open-source skills, and took **mechanisms, never text**. What survived: sub-agent scoping and the argument that a padded assignment is followed worse, not merely priced higher; grading criteria that have to be fixed before a run to count; an explicit reversibility line. What did not: anything about constructing an API call, anything naming a model generation, and persona and prompt-library material, which is a different genre this skill deliberately is not. One phrase here previously echoed caveman's own tagline too closely and was rewritten. Nothing is reproduced verbatim from any source.

Every addition was checked against a baseline run of the skill on four prompts, and guidance the baseline already satisfied was dropped rather than added. That is why there are still four checks and not six.

MIT licensed. Built by [Santiago Moya](https://github.com/stgomoyaa), who ran out of patience at commit 286 and wrote this instead.
