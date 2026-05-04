# SkillCraft

> **Anonymized submission** for the NeurIPS 2026 Evaluations & Datasets Track.
> Author names, institutional affiliations, repository owners, and project websites have been redacted.
> Please do not attempt to de-anonymize this submission.

## Project Overview

Real-world tool-using agents operate over long-horizon workflows with recurring structure. In this setting, strong behavior depends not only on calling atomic tools, but also on discovering, abstracting, and applying higher-level tool compositions.

**SkillCraft** is designed to explicitly evaluate this capability. The benchmark stress-tests whether agents can form and apply higher-level tool compositions (called **Skills**) under realistic, compositional tool-use scenarios.

Task difficulty is scaled along two axes:

- **Quantitative scaling**: increase the number of entities/items an agent must process.
- **Structural scaling**: compose subtasks into longer and more complex tool-use chains.

The accompanying protocol enables agents to compose atomic tools into executable skills, cache them, and apply them across tasks as a persistent skill library. In our paper's evaluation, this leads to substantial efficiency gains (up to 80% token reduction) while preserving strong task performance.

## Repository Layout (Reproduction-Relevant)

- `tasks/scaled_tasks/`: 126 evaluation tasks used in the paper (21 task families x 6 difficulty levels)
- `test_all_tasks.py`: main batch evaluation entrypoint
- `run.sh`: single-task runner
- `prefix.sh`: environment loading and runtime defaults
- `configs/mcp_servers/`: per-API MCP server configurations
- `deployment/`: mock services (Canvas LMS, Poste mail, WooCommerce) used by some task families
- `utils/`: agent runner, model wrappers, cost tracking, cross-model / cross-task harnesses
- `croissant.json`: Croissant metadata file (with Responsible-AI fields)

## Reproducibility Guide

### 1. Prerequisites

- Linux (recommended)
- Python 3.10+ (the lockfile pins 3.12.11)
- `uv` package manager
- Node.js 22+ with `npx` available (`scaled_tasks` launches the `filesystem` MCP server via `npx`)
- A valid LLM API endpoint/key (e.g., OpenRouter-compatible)
- Docker / Podman (recommended for environment consistency)

Install `uv` if needed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. Install dependencies

From repo root:

```bash
uv sync
```

### 3. Configure environment

Copy the example and fill in your API key:

```bash
cp configs/env.example .env
# edit .env: set SKILLCRAFT_OPENAI_API_KEY=...
```

`prefix.sh` will load `.env` automatically. Minimum required:

```bash
SKILLCRAFT_OPENAI_API_KEY=YOUR_API_KEY
SKILLCRAFT_OPENAI_BASE_URL=https://openrouter.ai/api/v1
SKILLCRAFT_MODEL=deepseek-v3.2-exp
SKILLCRAFT_PROVIDER=openrouter
```

### 4. Running the Pipeline

#### A. Single task

Base mode (no skill caching/reuse):

```bash
bash run.sh scaled_tasks/cat-facts-collector/e1 base --model deepseek-v3.2-exp --provider openrouter
```

Skill mode (with skill cache):

```bash
bash run.sh scaled_tasks/cat-facts-collector/e1 skill --model deepseek-v3.2-exp --provider openrouter
```

#### B. Complete evaluation (base + skill, all tasks)

Reproduces the main results in the paper:

```bash
uv run python test_all_tasks.py \
  --scaled-tasks \
  --mode base,skill \
  --model deepseek-v3.2-exp \
  --provider openrouter
```

#### C. Resume an interrupted run

```bash
uv run python test_all_tasks.py \
  --continue-run test_runs/run_YYYYMMDD_HHMMSS \
  --scaled-tasks \
  --mode base,skill \
  --model deepseek-v3.2-exp \
  --provider openrouter
```

> Supplementary analyses in the paper (hierarchical-mode comparison in Section 5.1, cross-task generalization in Section 5.2, cross-model skill reuse in Section 5.3 / Appendix D.4) use additional runners under `utils/`. They are out of scope for this README; the main Base vs. Skill results above are sufficient to verify the paper's headline claims.

## Expected Outputs

Each run produces a timestamped folder under `test_runs/`, including:

- `run_info.json` -- invocation metadata
- `test_results_<provider>_<model>.json` -- per-task outcomes
- `summary_<provider>_<model>.json` -- aggregate metrics (success / tokens / cost / turns / tool calls)
- `dumps_base_test/...` -- base-mode trajectories
- `dumps_skill_test/...` -- skill-mode trajectories

For result validation, check:

1. Mode-level summary in `summary_*.json`
2. Per-task `eval_res.json`
3. Per-task `traj_log.json` for completeness and tool-call traces

## License

CC-BY 4.0 -- see `LICENSE`. You are free to share and adapt the benchmark with appropriate attribution.

## Anonymity Notice

This repository has been scrubbed of author names, institutional affiliations, repository handles, project websites, internal team identifiers, and personal contact information. Internal codename references in the repository history have been replaced or removed for the same reason. If you spot any remaining de-anonymizing content, please **do not investigate or share it** -- report it via the OpenReview submission instead.
