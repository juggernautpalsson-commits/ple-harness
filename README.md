# ple-harness v0.1

Plan-work-verify agent harness for Google's Agent Harness on
Physical-Engineering's Last Exam (Kaggle `physical-engineerings-last-exam`,
$400k, Stage 1 deadline Jan 5, 2027).

Built on the google-research/ple `plan-verify` sample (Apache-2.0 — see
LICENSE): the same tool-calling loop runs three roles per task, each in its
own conversation:

1. **Planner** (read-only) — inspects the task's files, writes a plan: each
   deliverable with exact path/format, data/method, how to check it, and the
   hard constraints that must hold.
2. **Worker** — executes the plan, computes with code (never by hand), writes
   deliverables in the exact format asked (Word via python-docx, workbooks via
   openpyxl), re-opens each one to check it, and returns a summary as the
   final message.
3. **Verifier** (read-only, fresh conversation) — opens every deliverable,
   recomputes key numbers with code, checks hard constraints explicitly, and
   replies `PASS` or a numbered gap list. Gaps go back to the worker for up
   to 2 fix rounds while time remains.

Time budgets (of the 6-hour task limit): planner 10%, worker until 25%
remains, verifier/fix rounds until 4 minutes remain.

## Files

| File | Purpose |
|---|---|
| `serve.py` | The three-role loop, time budgets, role prompts |
| `taskbridge.py` | Serve protocol (shared code from the ple samples, unchanged) |
| `requirements.txt` | Pinned `openai`, `websockets`, `python-docx`, `openpyxl` |
| `submission.yaml` | Entry name `ple-harness`, target model `google-cloud/gemini-3.8-flash` |
| `LICENSE` | Apache-2.0 |
| `HARNESS-NOTES.md` | Design decisions, known gaps, v0.2 plan |

## Model route

- **Submission target:** `google-cloud/gemini-3.8-flash` (Google-sponsored,
  key supplied free at eval). Single target `model_api` committed.
- **Local dev:** `gemini/gemini-3.8-flash` on a Google AI Studio free-tier
  key, kept in `local.env` outside this repo (never committed). Generate
  dev `artifacts/` on the local model, switch `model_api` back to the
  `google-cloud/<id>` target before committing, and document the local model
  + judge setup in the Writeup (Kaggle page switch rule — see RULES.md §3).
- No API keys anywhere in this repo. The evaluator injects the model URL/key.

## Fair play (hard rules)

- No hard-coded answers, no task-specific branching — the same prompts run on
  every task, dev or hidden.
- The harness never reads dev-set graders or reference files, never reads its
  own `artifacts/`, never calls any host other than the model endpoint
  through the egress proxy. No web search, no provider-hosted tools.
- Committed `artifacts/` come only from `eval/run.py` on this harness.

## Checks

- `kit validate` after every structural change (works on this VM, $0).
- `kit check` + `eval/run.py --artifacts` before any submit — needs Docker +
  a run venue (not available on this VM yet).
- No runs yet. First measured number: `minimal`-equivalent smoke on 3 dev
  tasks (see `../dev-runs/README.md`) once the model route + venue are live.
