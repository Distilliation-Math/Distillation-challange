# SAIR Mathematics Distillation Challenge -- Equational Theories Stage 1

Competition entry for the [SAIR Mathematics Distillation Challenge](https://competition.sair.foundation/competitions/mathematics-distillation-challenge-equational-theories-stage1/overview), organized by [Terence Tao](https://terrytao.wordpress.com/2026/03/13/mathematics-distillation-challenge-equational-theories/) and the SAIR Foundation.

## Challenge

Build a compact prompt + cheatsheet (**max 10KB**) that maximizes LLM correctness on equational implication problems over magmas:

> *Given two equational laws, does Equation 1 imply Equation 2 over all magmas?*

A [magma](https://en.wikipedia.org/wiki/Magma_(algebra)) is a set with a single binary operation -- no associativity, commutativity, or identity assumed. The [Equational Theories Project](https://teorth.github.io/equational_theories/) catalogued 4,694 such laws and resolved all 22 million+ pairwise implications.

**Stage 1** asks for a TRUE/FALSE verdict only. **Stage 2** (May 2026) will require counterexamples, Lean proofs, or calibrated probabilities.

## Team

| Member | Role |
|--------|------|
| Andrea | Architecture, infrastructure, compute |
| Riccardo | Methodology, platform, submission |
| Tommaso | Research, mathematical content |

## Repository Structure

```
├── Plan.md                   # Project plan (phases, architecture, costs, timeline)
├── eval_harness.py           # Evaluation harness (JSONL → model API → CSV + accuracy)
├── config/
│   ├── models.yaml           # Model configuration for eval harness
│   └── prompts/              # Prompt templates (v0_baseline.txt, etc.)
├── cheatsheets/              # Versioned prompt+cheatsheet files
│   ├── v4_10KB_cheatsheet.md       # v4 cheatsheet (8.8KB)
│   ├── v4.1_10KB_cheatsheet.md     # v4.1 cheatsheet (10.2KB)
│   ├── v5.2_10KB_cheatsheet.md     # v5 structured-output experiment
│   ├── v5.5_10KB_cheatsheet.md     # v5.5 official-mode checkpoint
│   ├── v7.1_10KB_cheatsheet.md     # v7.1 cheatsheet (10.2KB)
│   ├── v7.4_10KB_cheatsheet.md     # Current best cheatsheet (10.2KB)
│   └── v.Opus-*_10KB_cheatsheet.md # Opus-informed iterations (Opus-1 / -2 / -2.1)
├── results/                  # Organized by batch: YYYYMMDD_HHMM_description/
│   ├── baselines/            # Baseline (no cheatsheet) results per model/dataset + SUMMARY.md
│   ├── iterations/           # Per-iteration results: v{N}_{model}_{dataset}.csv
│   ├── 20260406_2308_v4-all-models-hard/  # v4 sweep (7 models x 3 hard datasets)
│   ├── 20260410_v4.1-official/            # v4.1 official-mode sweep (SAIR evaluation_models.json)
│   ├── 20260412_v5.2_official/            # v5.2 official-mode sweep
│   ├── 20260417_v5.5_official_run3/       # v5.5 official-mode sweep (prior baseline)
│   ├── 20260419_v7.1_andrea/              # v7.1 official-mode sweep
│   ├── 20260419_v7.4/                     # v7.4 official-mode sweep (current best)
│   ├── 20260420_v7.5/                     # v7.5 official-mode sweep (regression test)
│   └── Opus_research/                     # All Opus-thread work (see note below)
│       ├── 20260411_opus-hard1/           # opus-solver raw-reasoning runs on hard1
│       ├── 20260413_v.Opus-1_official/    # v.Opus-1 cheatsheet sweep
│       ├── 20260413_v.Opus-2_official/    # v.Opus-2 cheatsheet sweep
│       ├── 20260413_v.Opus-2.1_official/  # v.Opus-2.1 cheatsheet sweep
│       └── 20260413_opus-analysis/        # Opus-vs-cheatsheet failure analysis + suggestions PDF
├── scripts/
│   ├── report_pdf.py          # Shared PDF report helper (see .claude/skills/generating-result-pdfs/)
│   ├── run_opus_benchmark.py  # Opus-solver benchmark driver (spawns claude -p --agent opus-solver)
│   └── refresh_sair_intel.py  # SAIR Zulip + contributor-network sync (48h workflow)
├── analysis/
│   └── error_taxonomy.py     # Failure-mode classifier for result JSONs
├── tests/                    # Pytest suite (parse_verdict, official_overrides, opus_agents)
├── training_data/            # Official problem sets (JSONL)
│   ├── normal.jsonl          # 1000 problems (500 TRUE / 500 FALSE)
│   ├── hard1.jsonl           # 69 problems (deduplicated hard set)
│   ├── hard2.jsonl           # 200 problems (100 TRUE / 100 FALSE)
│   └── hard3.jsonl           # 400 problems (195 TRUE / 205 FALSE)
├── extra_training_data/      # Community-contributed datasets
│   ├── SAIRCommunityBench_v1.jsonl         # 199 problems (order 4-6, harder distribution)
│   ├── Hard5_order5.jsonl                  # 654 problems (order-5 equations)
│   ├── full_2000_equations.jsonl           # 1999 problems (full community set)
│   └── verification_data_with_citations.jsonl  # 199 problems with Lean proof citations
├── research/
│   ├── equations.txt         # All 4694 equational laws
│   └── Raw_implication_graph.csv  # Per-equation implication statistics
├── blog_data/                # Community intelligence (see blog_data/README.md)
│   ├── cheatsheets/          # Community cheatsheets from SAIR contributor network
│   └── zulip/                # Zulip thread dumps, organized by stream
├── meeting_notes/            # Team meeting notes (Meetings_Notes_DD-MM-YY.md)
├── docs/                     # Reference material
└── .claude/                  # Project-scoped Claude Code config
    ├── agents/               # Custom subagents (opus-solver, opus-orchestrator, …)
    └── skills/               # Project skills (generating-result-pdfs, …)
```

**`results/Opus_research/` convention:** all Opus-thread artifacts (opus-solver raw-reasoning runs, v.Opus-N cheatsheet sweeps, opus-informed analysis PDFs) live under this subdirectory. Non-Opus sweeps stay at the `results/` root.

## Data Format

Each problem in the JSONL files has the following schema:

```json
{
  "id": "normal_0001",
  "eq1_id": 2918,
  "eq2_id": 1911,
  "equation1": "x = ((y * (x * y)) * z) * w",
  "equation2": "x = (y * (x * z)) * (y * w)",
  "answer": true
}
```

The full dataset is also available on HuggingFace: [SAIRfoundation/equational-theories-selected-problems](https://huggingface.co/datasets/SAIRfoundation/equational-theories-selected-problems).

## Architecture

All model inference runs in the cloud via [OpenRouter](https://openrouter.ai/). No local GPU required.

| Layer | Tool | Purpose |
|-------|------|---------|
| Orchestration | Claude Max 200K | Cheatsheet generation, compression, error analysis |
| Evaluation | `eval_harness.py` | Batch evaluation: substitute equations → call model → parse verdict → score |
| Inference | OpenRouter API | GPT-OSS-120b, GPT-OSS-20b, Gemma 4 31B, Llama 3.3 70B, DeepSeek V3.2, Gemini Flash Lite, Grok 4.1 Fast |

## Results

### v7.1 Cheatsheet -- Official-Mode Sweep (2026-04-19)

v7.1_10KB_cheatsheet.md (10.2KB) evaluated under `--official-mode` (provider pinning, `seed=0`, `max_tokens=8192`, reasoning_effort as per SAIR `evaluation_models.json`) on `gpt-oss-120b` and `gemma-4-31b` across hard1/hard2/hard3. Results in `results/20260419_v7.1_andrea/`. 0 parse errors on all 6 runs.

| Model | hard1 (69) | hard2 (200) | hard3 (400) | ALL (669) |
|-------|-----------|-------------|-------------|-----------|
| **gpt-oss-120b** | 54/69 (78.3%) | 145/200 (72.5%) | 257/400 (64.2%) | **456/669 (68.2%)** |
| **gemma-4-31b** | 56/69 (81.2%) | 149/200 (74.5%) | 255/400 (63.8%) | **460/669 (68.8%)** |

**v7.1 vs v5.5 (run3 baseline):**

| Model | DS | v5.5 Acc | v7.1 Acc | Δacc | v5.5 P1 | v7.1 P1 | ΔP1 |
|-------|-----|---------:|---------:|-----:|--------:|--------:|----:|
| gpt-oss-120b | hard1 | 73.9% | **78.3%** | +4.3 | 61.5% | **80.0%** | +18.5 |
| gpt-oss-120b | hard2 | 67.5% | **72.5%** | +5.0 | 73.3% | **80.8%** |  +7.5 |
| gpt-oss-120b | hard3 | **65.5%** | 64.2% | −1.3 | 73.2% | **75.0%** |  +1.8 |
| gemma-4-31b  | hard1 | 78.3% | **81.2%** | +2.9 | 73.7% | **78.9%** |  +5.3 |
| gemma-4-31b  | hard2 | 72.5% | **74.5%** | +2.0 | 74.7% | **81.0%** |  +6.3 |
| gemma-4-31b  | hard3 | 63.2% | **63.7%** | +0.5 | 70.3% | **73.1%** |  +2.8 |

v7.1 beats v5.5 on accuracy in 5/6 cells and on P1 (TRUE-precision) in all 6 cells.

**Key design wins in v7.1:**
- **Step 2.8 bare-source contradiction motifs** with broad catch-all rules (`rhsVars≥4 ∧ Lx=FALSE`) — drive a large fraction of TRUE recall on hard2/hard3. Pattern-specific motifs (narrow `rhsTotals` signatures) alone lose ~110 TRUE problems.
- **Feature-vector discipline**: mandatory 7-feature write-out (rhsVars, Lx, Rx, xTop, topShape, rhsTotals, xCount) before motif lookup — forces explicit computation rather than rule-name guessing.
- **Hard-stop on FALSE rule / counterexample fire** — blocks GPT-OSS-120b confabulation pattern.
- **Spine isolation** (3.0): pure-spine depth divisibility — eliminates a well-defined false-TRUE class without touching positives.

### v7.4 Cheatsheet -- Official-Mode Sweep (2026-04-19) ← Submission Candidate

v7.4_10KB_cheatsheet.md (10.2KB) evaluated under `--official-mode` on `gpt-oss-120b` and `gemma-4-31b` across hard1/hard2/hard3. Results in `results/20260419_v7.4/`. 0 parse errors on all 6 runs.

| Model | hard1 (69) | hard2 (200) | hard3 (400) | ALL (669) |
|-------|-----------|-------------|-------------|-----------|
| **gpt-oss-120b** | 56/69 (81.2%) | 145/200 (72.5%) | 256/400 (64.0%) | **457/669 (68.3%)** |
| **gemma-4-31b** | 55/69 (79.7%) | 143/200 (71.5%) | 256/400 (64.0%) | **454/669 (67.9%)** |

**Confusion matrix (hard1+hard2+hard3 combined):**

| Model | TP | FP | TN | FN | PE | TRUE-rec | FALSE-rec | TRUE-prec |
|-------|---:|---:|---:|---:|---:|---------:|----------:|----------:|
| gpt-oss-120b | 129 | 22 | 328 | 190 | 0 | 40.4% | 93.7% | 85.4% |
| gemma-4-31b | 136 | 32 | 318 | 183 | 0 | 42.6% | 90.9% | 81.0% |

**v7.4 vs v7.1:**

| Model | DS | v7.1 Acc | v7.4 Acc | Δacc | v7.1 FP | v7.4 FP | ΔFP |
|-------|-----|---------:|---------:|-----:|--------:|--------:|----:|
| gpt-oss-120b | hard1 | 78.3% | **81.2%** | +2.9 | 3 | 3 | 0 |
| gpt-oss-120b | hard2 | 72.5% | 72.5% | 0.0 | 14 | **9** | −5 |
| gpt-oss-120b | hard3 | 64.2% | 64.0% | −0.2 | 26 | **10** | −16 |
| gemma-4-31b  | hard1 | **81.2%** | 79.7% | −1.4 | 4 | 4 | 0 |
| gemma-4-31b  | hard2 | **74.5%** | 71.5% | −3.0 | 15 | **14** | −1 |
| gemma-4-31b  | hard3 | 63.7% | **64.0%** | +0.3 | 29 | **14** | −15 |

v7.4 trades a small amount of TRUE-recall for substantially fewer false positives (−37 FP combined across both models vs v7.1). This makes it more robust to distribution shifts in the hidden test set.

**Key design changes from v7.1 to v7.4:**
- **Broad catch-all C1 restored** (`rv≥4, Lx=FALSE, Rx=FALSE`) — essential for TRUE recall; narrowing to pattern-specific `rhsTotals` in v7.2 lost ~110 TRUE problems.
- **Additional motifs C6-C11** — `xCount=0` safety net, m-m topShape patterns, higher-order rTot signatures.
- **Bidirectional hard-stop** ("Once ANY rule fires, emit VERDICT and STOP") — prevents GPT confabulation pattern where correct counterexamples are overridden by fluent wrong proofs.
- **Removed Step 2.11** (two-step exact instance) — was an early-exit trap that blocked the cascade from reaching motifs.
- **Abbreviated feature names** (`rv`, `ts`, `rTot`) — saves bytes for additional motifs.

### v4.1 Cheatsheet -- 4-Model Hard Sweep (2026-04-09)

v4.1_10KB_cheatsheet.md (10.2KB) evaluated on 4 models x hard1/hard2/hard3 via OpenRouter. All parse errors resolved. Full report with confusion matrices: [`results/20260409_v4.1-retry2/v4.1_results.pdf`](results/20260409_v4.1-retry2/v4.1_results.pdf).

| Model | hard1 (69) | hard2 (200) | hard3 (400) | ALL (669) | Bias |
|-------|-----------|-------------|-------------|-----------|------|
| **grok-4.1-fast** | 53/69 (76.8%) | **160/200 (80.0%)** | 237/400 (59.2%) | **450/669 (67.3%)** | Balanced |
| **gpt-oss-120b** | **54/69 (78.3%)** | 150/200 (75.0%) | **240/400 (60.0%)** | 444/669 (66.4%) | Slight TRUE |
| gpt-oss-20b | 48/69 (69.6%) | 140/200 (70.0%) | 234/400 (58.5%) | 422/669 (63.1%) | Moderate TRUE |
| gemma-4-31b | 51/69 (73.9%) | 117/200 (58.5%) | 226/399 (56.6%) | 394/668 (59.0%) | Extreme FALSE |

**Key improvements over v4:**
- **GPT-OSS-120b: +34pp hard1, +22pp hard2, +14pp hard3** -- was 43.9%/52.6%/46.1% in v4. Fixed by `temperature=1.0` + `max_tokens=65536`
- **GPT-OSS-20b: 83 parse errors → 0** -- `temperature=0` caused OpenRouter to silently drop requests for reasoning models
- **Harness fixes:** per-model params in `config/models.yaml`, per-request 180s timeout, null body guard, empty response retry

### v4 Cheatsheet -- Full Hard Sweep (2026-04-06)

v4_10KB_cheatsheet.md (8.8KB) evaluated on all 7 models x hard1/hard2/hard3 via OpenRouter. Full results with JSON reasoning in `results/20260406_2308_v4-all-models-hard/`.

| Model | hard1 (69) | hard2 (200) | hard3 (400) | Parse Err | Bias Pattern |
|-------|-----------|-------------|-------------|-----------|--------------|
| **grok-4.1-fast** | **53/68 (77.9%)** | **153/200 (76.5%)** | **242/400 (60.5%)** | 1 | Balanced |
| gemma-4-31b | 43/69 (62.3%) | 145/197 (73.6%) | 202/385 (52.5%) | 18 | TRUE bias (confab) |
| gemini-flash-lite | 51/69 (73.9%) | 104/200 (52.0%) | 225/400 (56.2%) | 0 | FALSE bias |
| deepseek-v3.2 | 45/69 (65.2%) | 101/200 (50.5%) | 206/400 (51.5%) | 0 | Total FALSE (0% TRUE recall) |
| llama-3.3-70b | 45/69 (65.2%) | 100/200 (50.0%) | 206/400 (51.5%) | 0 | Total FALSE (0% TRUE recall) |
| gpt-oss-20b | 38/65 (58.5%) | 76/174 (43.7%) | 179/358 (50.0%) | 72 | TRUE bias + parse err |
| gpt-oss-120b | 29/66 (43.9%) | 102/194 (52.6%) | 170/369 (46.1%) | 40 | Extreme TRUE bias |

### Baseline (no cheatsheet)

| Model | normal (1000) | hard1 (69) | hard2 (200) | hard3 (400) |
|-------|:---:|:---:|:---:|:---:|
| grok-4.1-fast | 32.0% | 55.1% | 39.0% | 62.0% |
| gpt-oss-120b | 51.9% | 50.7% | 45.5% | 46.5% |
| llama-3.3-70b | 36.7% | 46.4% | 36.5% | 34.8% |
| gemini-flash-lite | 49.8% | 65.2% | 50.0% | 51.2% |

### v1 Cheatsheet (historical, 4.6KB)

| Model | normal | Baseline | Delta |
|---|---|---|---|
| **grok-4.1-fast** | 840/1000 (84.0%) | 320/1000 (32.0%) | **+52.0pp** |
| gpt-oss-120b | 629/1000 (62.9%) | 519/1000 (51.9%) | +11.0pp |
| llama-3.3-70b | 500/1000 (50.0%) | 367/1000 (36.7%) | +13.3pp |
| gemini-flash-lite | 502/1000 (50.2%) | 498/1000 (49.8%) | +0.4pp |

## Key Resources

**Competition:**
- [Competition page](https://competition.sair.foundation/competitions/mathematics-distillation-challenge-equational-theories-stage1/overview)
- [SAIR Playground](https://competition.sair.foundation/) -- test prompts against models (10 credits/day)
- [SAIR Zulip](https://zulip.sair.foundation/) -- community discussion forum

**Equational Theories Project:**
- [ETP main page](https://teorth.github.io/equational_theories/) -- full implication graph and equation explorer
- [Graphiti visualization](https://teorth.github.io/equational_theories/graphiti/) -- 22M-edge implication graph
- [Tao's AlphaEvolve predictors](https://github.com/teorth/equational_theories/tree/main/scripts/predictor) -- Python-based implication predictors
- [Hard implications blueprint](https://teorth.github.io/equational_theories/blueprint/hard-implications-chapter.html) -- annotated hardest cases
- [Order-5 ETP branch](https://github.com/vlad902/equational_theories/tree/order5) -- extended equation data

**Papers:**
- [Honda et al. (2025)](https://arxiv.org/abs/2509.20820) -- "Distilling Many-Shot In-Context Learning into a Cheat Sheet"
- [ETP paper (2025)](https://arxiv.org/abs/2512.07087) -- "The Equational Theories Project: Advancing Collaborative Mathematical Research at Scale"
- [Vampire on ETP (2025)](https://arxiv.org/abs/2508.15856) -- ATP performance analysis on these implications

**Tools:**
- [CyberAgentAILab/cheat-sheet-icl](https://github.com/CyberAgentAILab/cheat-sheet-icl) -- reference implementation of the distillation paper
- [Knuckledragger](https://github.com/philzook58/knuckledragger) -- Python proof assistant with Z3/Vampire/Prover9 ([blog](https://www.philipzucker.com/tao_algebra/))
- [adamtopaz equational_dataset](https://huggingface.co/datasets/adamtopaz/equational_dataset) -- HuggingFace ETP implications dataset

## Timeline

| Date | Milestone | Status |
|------|-----------|--------|
| Mar 30 | Baselines + eval harness complete | Done |
| Mar 30 | v1 cheatsheet evaluated (84% grok normal) | Done |
| Apr 6 | v4 full sweep: 7 models x 3 hard datasets (21 runs) | Done |
| Apr 6 | JSON failure analysis + community intel review | Done |
| Apr 7 | max_tokens increased to 16384, PPTX summary generated | Done |
| Apr 8-9 | v4.1 sweep: temp=1.0 fix, harness hardening, 4 models x 3 hard | Done |
| Apr 9+ | v5 cheatsheet: VERDICT-first, feature-first, spine isolation | Done |
| Apr 10 | Evaluation models announced by SAIR | Done |
| Apr 11 | Model-specific optimization begins | Done |
| Apr 17 | v5.5 official-mode sweep (prior baseline) | Done |
| Apr 19 | v7.1 official-mode sweep: step-2.8 motif expansion, feature-vector discipline | Done |
| Apr 19 | v7.2–v7.4 iteration: motif tuning, hard-stop experiments, C1 guard | Done |
| Apr 20 | v7.5 regression test (asymmetric hard-stop); v7.4 selected as submission | Done |
| **Apr 20** | **Submission deadline (23:59 AoE)** | |
