# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

SAIR Mathematics Distillation Challenge -- Equational Theories Stage 1. Build a compact prompt+cheatsheet (max 10,240 bytes) that maximizes LLM correctness on "Does Equation 1 imply Equation 2 over all magmas?" (TRUE/FALSE). The submission is one complete prompt containing `{{ equation1 }}` and `{{ equation2 }}` placeholders; the harness sends it as the only user message (no system prompt).

**Status:** Stage 1 closed 2026-04-20 23:59 AoE; `cheatsheets/v7.4_10KB_cheatsheet.md` was selected as the submission. Post-deadline hard5 and non-official runs followed on 2026-04-21. **Paused since 2026-10-19:** the intel-refresh workflow is disabled (`gh workflow enable sair-intel-refresh.yml` resumes it). README: Stage 2 (May 2026) requires counterexamples, Lean proofs, or calibrated probabilities. Team: Andrea (architecture/infra/compute), Riccardo (methodology/platform/submission), Tommaso (research/math). `meeting_notes/` are in Italian and record most design decisions.

## Commands

No requirements file; Python 3.10+. Harness deps: `pip install aiohttp certifi pyyaml python-dotenv` (plus `requests` for the intel refresh, `fpdf2` for PDF reports). `OPENROUTER_API_KEY` (and `ZULIP_EMAIL`/`ZULIP_API_KEY` for the refresh) are read from `.env` at the repo root.

```bash
# Official-mode eval -- how every comparable sweep is run (mirrors SAIR's evaluation_models.json)
python3 eval_harness.py --model gpt-oss-120b --backend openrouter --official-mode \
    --prompt cheatsheets/v7.4_10KB_cheatsheet.md --data hard2 --output-dir results/YYYYMMDD_<desc>/

# No API calls: loads data, renders the prompt, warns if >10,240 bytes; writes no files
python3 eval_harness.py --model gemma-4-31b --backend openrouter --prompt <file> --data hard1 --dry-run --limit 5

# Tests are plain-assert scripts with their own runner (pytest is not installed)
for t in tests/test_*.py; do python3 "$t"; done
python3 -c "import sys; sys.path.insert(0, 'tests'); import test_parse_verdict as t; t.test_verdict_true()"   # single test

python3 analysis/error_taxonomy.py results/<batch>/*.csv --compare      # failure-mode taxonomy
python3 scripts/run_opus_benchmark.py --dataset hard1 --limit 3 --parallel 3 \
    --output-dir results/Opus_research/<YYYYMMDD>_<desc>/                # one `claude -p --agent opus-solver` per problem
python3 scripts/refresh_sair_intel.py                                   # Zulip + contributor-network sync into Blog_data/
```

## Architecture

**Path casing:** git, `config/models.yaml`, every script, and CI use `Training_data/`, `Extra_training_data/`, `Research/`, `Blog_data/`. The Mac's case-insensitive filesystem (`core.ignorecase=true`) also resolves lowercase paths, which hid a lowercase on-disk copy until 2026-10-19. Always write the capitalized form: lowercase paths break on Linux and in CI.

**`eval_harness.py`** (single file, async aiohttp): load JSONL → substitute `{{ equation1 }}`/`{{ equation2 }}` (spaced or not) → OpenRouter or local Ollama → `parse_verdict` → CSV + JSON.
- **Parameter precedence** (`config/models.yaml`): `defaults` < per-model `params` < `official_params` (only with `--official-mode`). CLI `--temperature`/`--max-tokens` only overwrite `defaults`, so they are **silently ignored** for gpt-oss-120b, gemma-4-31b, and gpt-oss-20b, which set both in `params`. Edit models.yaml instead. `--concurrency` is likewise replaced by `official_params.concurrency` (gpt-oss-120b: 7), or capped at 3 whenever a reasoning effort is sent.
- **`--official-mode`** (OpenRouter only) applies `official_params`: pinned provider+quantization with `allow_fallbacks: false`, reasoning effort, `seed: 0`, `max_tokens: 8192`. It never changes temperature. Only gpt-oss-120b, gemma-4-31b, and llama-3.3-70b have this block; they are SAIR's three Stage 1 eval models. `--official-fallbacks` appends `fallback_providers` and widens the quantization filter, which diverges from SAIR's spec.
- **Non-official runs** send `params.reasoning_effort` (currently `high` for gpt-oss-120b and gemma-4-31b).
- **Retries:** HTTP 429/5xx, transient `error.code` inside 200 bodies, and empty `content` get up to 5 attempts (backoff 5/15/30/60/120 s, 300 s per-call timeout; `--call-timeout` help text still says 180). On OpenRouter, a problem whose `content` stays empty is recorded as a parse error. Only the Ollama path falls back to parsing the `reasoning` field.
- **`parse_verdict`** is a port of SAIR's official `judge.py`, and `tests/test_parse_verdict.py` mirrors their `test_judge.py`, so keep the two in sync. Tiers, highest first: `\boxed{}`, then labeled markers (`VERDICT:`/`ANSWER:`/`FINAL ANSWER:`/`RESULT:`/`OUTPUT_RESULT:`/`\text{}`), then a bare TRUE/FALSE on the first or last non-empty line. **Within a tier the last occurrence wins.** Markdown is stripped, and instruction echoes ("VERDICT: TRUE or FALSE") are ignored.
- **Scoring:** accuracy = correct / (total − parse_errors). Parse errors are excluded from the denominator.
- **Output:** `<--output-dir or results/YYYYMMDD_HHMM>/<model>_<backend>_<data>_<YYYYMMDD_HHMMSS>.csv` (response cut to 200 chars), plus a same-stem `.json` with summary, confusion counts, and full `content` + `reasoning` per problem. Use the JSON for failure analysis. Pass a `datasets` key to `--data`: a path gets interpolated into the filename and creates nested dirs (as in `results/20260419_v7.3_full_2000/`).
- **Code that shares this contract:** `scripts/run_opus_benchmark.py` imports `parse_verdict` and writes the same JSON schema. `analysis/error_taxonomy.py` reads the CSVs plus `Research/Raw_implication_graph.csv`. `tests/test_opus_agents.py` asserts content markers in `.claude/agents/opus-*.md` and in the driver, so change them together.

**Data:** fields are `id`, `equation1`, `equation2`, `answer`, and `eq1_id`/`eq2_id`, where line n of `Research/equations.txt` is equation n. Equations use `*`.
- `Training_data/`:
  - `normal` 1000 (500 TRUE / 500 FALSE)
  - `hard1` 69 (24/45), the deduplicated form of `hard.jsonl` (200)
  - `hard2` 200 (100/100)
  - `hard3` 400 (195/205)
- `Extra_training_data/`:
  - `community` = `SAIRCommunityBench_v1.jsonl` 200 (100/100, order 4-6)
  - `hard5` = `Hard5_order5.jsonl` 654 (327/327; uses `equation1_id`/`equation2_id`)
  - `full_2000_equations.jsonl` 2000 (1000/1000; no dataset key)
  - `verification_data_with_citations.jsonl` 200, the community bench with Lean proof/countermodel citations
- `Research/`:
  - `equations.txt`: all 4694 laws, using `◇`
  - `Raw_implication_graph.csv`: implication counts for 1415 laws, one per equivalence class
  - `v7.1_deep_research_synthesis.md`: the literature and failure-bucket analysis behind v7.3+
  - ETP and Honda et al. PDFs

**Results:**
- One directory per batch: `results/YYYYMMDD[_HHMM]_<desc>/`.
- Opus-thread work (opus-solver runs, v.Opus-N sweeps, Opus analyses) goes under `results/Opus_research/`, never at the `results/` root.
- `results/Quality_Assurance/` holds the run-to-run and SAIR-vs-local reproducibility studies.
- `results/baselines/` holds the no-cheatsheet runs plus `SUMMARY.md`. `results/iterations/` is legacy (v1 only).
- PDFs: write a `render.py` beside the data that imports `scripts/report_pdf.py` (skill: `generating-result-pdfs`). Do not add new `gen_pdf.py` copies.

**Automation:**
- `.github/workflows/sair-intel-refresh.yml` is **disabled** since 2026-10-19 (re-enable: `gh workflow enable sair-intel-refresh.yml`). When enabled, it runs the refresh every 48h and force-updates one rolling PR on the fixed branch `sair-intel-refresh`. Never reintroduce a per-run branch suffix: it opened a new PR every run (90+ piled up unmerged). Local `Blog_data/` only reflects merged refreshes; the last (PR #107) covers community data up to 2026-10-19.
- Entry points into the community data: `Blog_data/cheatsheets/INDEX.md` and `Blog_data/zulip/INDEX.md`.
- Project agents in `.claude/agents/`:
  - `opus-solver`: raw Opus reasoning with no cheatsheet
  - `opus-orchestrator`: thin wrapper around the benchmark driver, because subagents cannot spawn subagents
  - `sair-community-intel-updater`
  - `code-review-sentinel`

## Reference Numbers

v7.4, official mode, `results/20260419_v7.4/`, 0 parse errors:

| Model | hard1 | hard2 | hard3 | ALL (669) | TRUE-rec | FALSE-rec |
|---|---|---|---|---|---|---|
| gpt-oss-120b | 81.2% | 72.5% | 64.0% | 68.3% | 40.4% | 93.7% |
| gemma-4-31b | 79.7% | 71.5% | 64.0% | 67.9% | 42.6% | 90.9% |

- **Pre-edit version:** that sweep scored the 10,220-byte version from b35cc1f (`git show b35cc1f:cheatsheets/v7.4_10KB_cheatsheet.md`). Commit 1da98ff later edited the file down to 10,212 bytes, and `20260421_v7.4_*` used the edited file. That edit left the §2.8 C5 example inconsistent (Eq1 `x = (y*(z*x)*z)*z` vs `P = (y*(z*x))*z`). v7.4 is frozen as the submitted artifact: do not edit it; iterate in a new version file.
- **Other v7.4 runs:**
  - `20260419_v7.4_run2/`: a repeat run
  - `20260421_v7.4_hard5/`: gpt 66.1%, gemma 63.3%
  - `20260421_v7.4_non_official/` (reasoning `high`, no provider pin): hard2 76.5% / 79.5% (gpt/gemma), hard3 61.8% / 63.0%
- **Models covered:** v7.x sweeps cover only gpt-oss-120b and gemma-4-31b. Llama was deprioritized at the 12-04 meeting: under official mode it answers FALSE on almost everything (v5.5: 0-1 TP per dataset).
- **Noise:**
  - Identical official runs flip 17-28% of gpt-oss-120b answers; hard2 accuracy moved 6pp between two runs. Gemma flips 1-6% and llama ~0%.
  - SAIR-platform and local runs agree within ~2-3pp on aggregate accuracy, but differ on 28-36% of individual problems.
  - Treat deltas under ~3pp as noise. Compare FP/FN counts and use repeat runs.

## Key Domain Knowledge

- **Magma:** a set with one binary operation and no axioms (no associativity, commutativity, or identity).
- **ETP:** the Equational Theories Project resolved all 22M+ implications among the 4694 laws. Laws equivalent to `x = y` force a singleton magma, so they imply everything.
- **Official eval** (SAIR judge repo + Zulip): temperature 0.0, `seed: 0`, `max_tokens: 8192`, pinned providers.
  - gpt-oss-120b: DeepInfra bf16, reasoning `low`.
  - llama-3.3-70b: DeepInfra fp8, reasoning off.
  - gemma-4-31b: Novita bf16, reasoning disabled by SAIR because it exhausted the 8192 tokens before emitting content. Our `official_params` omits reasoning, which matches.
- **Hidden test set:** 50/50 TRUE/FALSE (Tao, on Zulip), so a default-FALSE fallback is calibrated. Tao also floated adding order-5 equations, which is why `hard5` matters.
- **Temperature:** gpt-oss-20b keeps `temperature: 1.0`; at 0 it silently returned empty responses on OpenRouter. gpt-oss-120b and gemma-4-31b have run at 0.0 since 2026-04-17, with 0 empty responses in official runs.
- **Failure modes** (v4 analysis):
  - **Confabulation is GPT's #1 failure:** GPT-OSS-120b finds a correct counterexample, then talks itself out of it with a fluent wrong proof.
  - GPT-OSS-20b misfires the singleton/collapse heuristic (58% of its failures).
  - DeepSeek, Llama, and Gemini answer FALSE on 99%+ of problems and treat the cheatsheet as a lookup table.
  - Gemini invents rules that are not listed ("rightmost exclusion", "count exclusion"), which account for 53% of its failures.
- **Token budget:** at 1024 max_tokens, ~90% of early GPT-120b failures were truncations. Official mode caps output at 8192 tokens including reasoning. Non-official `params` allow 125K (gpt-oss-120b), 256K (gemma), and 65K (gpt-oss-20b).

## Cheatsheet Design Rules (v7.x lessons)

- **Deterministic classifier, not free-form proving.** Numbered steps, STOP at the first decisive route, in this order:
  1. TRUE routes (§2)
  2. Syntactic FALSE invariants (§3)
  3. Small finite witness magmas (§4)
  4. Default FALSE, whose REASONING must say "default false fallback"

  TRUE recall (~40%) is the bottleneck. Under v7.4, ~60% of gpt-oss hard2 answers were the default fallback, and 75% of missed implications have a bare source `x = P`.
- **Bidirectional hard-stop:** "Once ANY rule fires a verdict (TRUE or FALSE), emit VERDICT immediately and STOP." This blocks GPT confabulation. v7.5's FALSE-only hard-stop did not beat v7.4: hard2 −3.0pp (gpt) / −1.5pp (gemma), with more FPs.
- **Output format:** v7.x emits `REASONING:`, then `VERDICT:`, then `PROOF:`. Putting the verdict first is not required, because the grader finds it anywhere, and v7.x runs show 0-3 parse errors. Any later `VERDICT:`, `ANSWER:`, or `\boxed{}` overrides an earlier one, so never prompt the model to restate or quote a verdict marker.
- **Feature vector before motifs (§2.8):** make the model write all 7 features (`rv, Lx, Rx, xTop, ts, rTot, xCount`) before comparing against the C-motifs. Otherwise it pattern-matches rule names. A motif non-match never implies FALSE.
- **Keep C1 broad, but guarded:**
  - C1 (`rv≥4, Lx=FALSE, Rx=FALSE`) drives most TRUE recall on hard2/hard3. v7.2 narrowed it to specific `rTot` signatures and lost ~110 TRUE answers across 6 model×dataset cells, about 80% of v7.2's regression.
  - Ground truth over all 3,822 unique dataset problems: v7.4's **unguarded C1 fires on 5 FALSE problems** (hard3_0165-0167, 0241-0242; sources `x = (y*x)*(x*(z*w))` and `x = ((y*z)*x)*(x*w)`).
  - Adding the v7.3/v7.5 guard `NOT (xTop=both AND ts=m-m)` gives 352 TRUE / 0 FALSE. C2-C11 fire on 0 FALSE problems.
  - v7.1's broader catch-all `rv≥4, Lx=FALSE` (Rx free) fires on 50 FALSE problems. Do not bring it back.
  - Validate every new motif against the labeled data, or with a finite-magma search, before shipping.
- **Spine isolation (§3.0, McKenna):** a pure left-spine (right-spine) bare law implies only pure left-spine (right-spine) laws of equal or greater depth. Within the same spine type, depth n must divide depth m. Validated on 1.54M pairs with zero exceptions; it fixed +32 hard3 problems.
- **One change at a time:** V6 combined default-TRUE with heavy anti-confabulation guardrails. That hurt the prompt's readability for the model, caused parse errors, and the version was abandoned. V7 restarted from v5.5.
- **Generalize:** test on hard1/2/3, `community`, `hard5`, and full_2000. Tuning to one set does not transfer: `cheatsheets/EQT01-000052_hard3_overfitted.md` scores 77.8% on hard3 but 42.0% on hard1 with gpt-oss-120b.
- **Model framing:**
  - Llama: minimal task framing, no "mathematician persona".
  - GPT: symbolic solver format with rule locks.
  - Gemini: an explicit "do not invent rules not listed here".

## External Resources

- ETP raw implications: teorth.github.io/equational_theories/implications/ (Graphiti: teorth.github.io/equational_theories/graphiti/)
- SAIR judge (official parser + `evaluation_models.json`): github.com/SAIRcompetition/equational-theories-stage1-judge
- Tao's AlphaEvolve predictors: github.com/teorth/equational_theories/tree/main/scripts/predictor
- Cheat-sheet ICL reference: github.com/CyberAgentAILab/cheat-sheet-icl. Knuckledragger (Z3/Vampire/Prover9): github.com/philzook58/knuckledragger
- HF datasets: SAIRfoundation/equational-theories-selected-problems, adamtopaz/equational_dataset. Order-5 ETP: github.com/vlad902/equational_theories/tree/order5
- SAIR Playground: competition.sair.foundation (10 credits/day). Zulip: zulip.sair.foundation
