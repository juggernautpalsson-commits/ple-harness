# Harness design notes — ple-harness v0.1 (Nov 14, 2026)

## Why plan-verify, not minimal

The `minimal` sample is one loop: one conversation, no independent check.
The dev set punishes exactly that shape — 12 `checks` tasks are calculation
packages where a wrong number is a wrong score, and 14 `rubric` tasks are
judged on the deliverable + final message. A second, fresh pair of eyes
(the verifier, with no access to the worker's reasoning) is the cheapest
structural fix for "wrote it once, never verified" (DEV-SET.md gap 5,
taxonomy F5). Cost: roughly 2–3× the model calls of `minimal` on the same
task — a real cost, and the score-vs-cost plot (Writeup criterion 4) must
show whether the verifier earns its tokens. If it doesn't, v0.2 cuts it to
one fix round or drops it for `checks` tasks only... no — the verifier
stays or goes as a whole, by measured ablation (A→B on the dev set), never
by hunch. No task-specific branching: the same loop runs on every task.

## Why Gemini 3.8 Flash as the target

- It is one of the 8 sponsored Stage 2 models, so the submitted
  `model_api` is legal and the eval key works for it.
- It is also the free-tier dev model (`gemini/gemini-3.8-flash` on an AI
  Studio key, no card), so dev runs and the submitted target are the same
  model family — the switch rule stays honest and the dev scores are a fair
  lower bound, not a different model's numbers.
- Trade-off, stated plainly: Flash is weaker than Opus 5.5 / Argon / 3.1 Pro
  on reasoning. The paper's results say model choice matters more than
  harness choice. If a second family becomes available for dev (personal
  subscription, open-weight), cross-model comparison (Writeup criterion 3)
  may move the target to a stronger model — decided by measured
  score-vs-cost, not by preference.

## Known gaps in v0.1 (all filed, none hidden)

1. **No MCP tools.** The 12 world-output tasks grade what the task's MCP
   servers / `/app/tools` apps produce. v0.1 only has bash/read/write, so
   `kit check` will warn at the `mcp` stage and world-output tasks will
   likely underperform. v0.2 ports the MCP client pattern from the
   `custom-openai` sample.
2. **No streaming.** Long model calls are non-streaming; the egress proxy
   closes connections idle 30 minutes. `custom-openai` streams. v0.2.
3. **No context compaction.** On context-full, v0.1 only trims old tool
   results (inherited from the samples). Long 6-hour tasks will hit this.
   v0.2: compact into notes (see `custom-openai`).
4. **No IFC writer.** `data-center-site-model` and the two MEP routing
   tasks need IFC4 output. `ifcopenshell` is heavy for the 2 GiB build
   cap — evaluate size before adding; the worker can also generate IFC via
   a small hand-written writer for the narrow shapes these tasks need.
   Deferred to v0.2, decided by measurement on those three tasks.
5. **Verifier cost.** See above — ablate it.

## What v0.1 does NOT do (fair play)

- No task-specific code paths. No reading graders/reference files. No
  reading `artifacts/`. No external hosts. No keys in the repo.
- The prompts contain no task names, no task IDs, no answers.

## Version log

- **v0.1 (Nov 14, 2026):** plan-verify structure + engineering role
  prompts + python-docx/openpyxl. Static only; `kit validate` green.
  No runs yet (blocked: model route + run venue).
