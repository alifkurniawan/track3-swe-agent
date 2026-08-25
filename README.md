# Take-Home Submission

## Tracks Completed

- **Track 3: SWE Agent** ✅

## How to Reproduce

### Prerequisites
```bash
git clone https://github.com/SWE-agent/mini-swe-agent.git
cd mini-swe-agent
uv pip install -e .
uv add swebench sb-cli  # sb-cli optional, local eval used instead
export OPENAI_API_KEY="<your-key>"   # endpoint/model configurable via --model flag below
```
Docker with `linux/amd64` emulation enabled is required on Apple Silicon (Docker Desktop → Settings → General → "Use Rosetta for x86/amd64 emulation").

Copy `swebench_custom.yaml` and `platform_override.yaml` (see repo root) into the `mini-swe-agent` working directory before running.

### Task set (pinned, 4 instances from SWE-bench Verified / django-django)
```
django__django-10554
django__django-10914
django__django-11099
django__django-11141
```
(Full list saved in `task_subset.txt`.)

### Model & config (pinned)
- Model: `gpt-5-nano` (OpenAI, via LiteLLM — swap with `--model <other>` to reproduce on a different endpoint)
- Temperature: `0`
- `mini-swe-agent` version: `2.4.6`

### Command 1 — Reference agent baseline (regenerates baseline numbers)
```bash
uv run mini-extra swebench \
  --subset verified --split test \
  --filter '^(django__django-10554|django__django-10914|django__django-11099|django__django-11141)$' \
  --model gpt-5-nano --environment-class docker --workers 1 \
  -c swebench.yaml -c platform_override.yaml -c model.model_kwargs.temperature=0 \
  -o ./results/baseline

uv run swebench eval verified -p ./results/baseline/preds.json --run-id baseline-eval -j 2
```

### Command 2 — This submission's harness (regenerates improved-agent numbers)
```bash
uv run mini-extra swebench \
  --subset verified --split test \
  --filter '^(django__django-10554|django__django-10914|django__django-11099|django__django-11141)$' \
  --model gpt-5-nano --environment-class docker --workers 2 \
  -c swebench.yaml -c swebench_custom.yaml -c platform_override.yaml -c model.model_kwargs.temperature=0 \
  -o ./results/harness

uv run swebench eval verified -p ./results/harness/preds.json --run-id harness-eval -j 2
```

Full run logs (`minisweagent.log`), per-instance trajectories (`.traj.json`), predictions (`preds.json`), and evaluation reports (`*.eval.json`) for both runs are included under `results/` for audit without needing to re-run or hold an API key.

## Design Document

See [`design_doc.md`](./design_doc.md) — covers what was built, rationale, results, contamination check, known limitations, and next steps.

## Time Investment & Scope Reductions

- **Time invested:** ~10 hours total, compressed into a single working session (task originally allotted ~1 week, executed near the deadline).
- **Scope reduction:** Used 4 of the ~8 recommended SWE-bench Verified instances, all from `django/django`, to keep Docker build overhead manageable on Apple Silicon within the available time. This is documented and reasoned about in `design_doc.md` §2.
- **Discarded run:** An initial baseline attempt using `gpt-4o-mini` failed due to a Docker platform-flag infrastructure bug (not an agent/model failure) and was discarded rather than reported as a result — see `design_doc.md` §2 for full transparency on this.
