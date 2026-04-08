# GPT-OSS Failure Analysis — v4 cheatsheet, 16k max_tokens

Run date: 2026-04-08. Models evaluated via OpenRouter on hard1 (69), hard2 (200), hard3 (400).

---

## GPT-OSS-120b — 413/669 correct (61.7%)

| Dataset | Accuracy | Bias |
|---------|----------|------|
| hard1 | 46/69 (66.7%) | Slightly FALSE-leaning (43% TRUE) |
| hard2 | 138/200 (69.0%) | TRUE-leaning (56% TRUE) |
| hard3 | 229/400 (57.2%) | TRUE-leaning (55% TRUE) |

### Failure breakdown (256 total)

- **57% false positives** (147) — says TRUE when answer is FALSE
- **41% false negatives** (104) — says FALSE when answer is TRUE
- **2% timeouts/errors** (5)

### False positive causes (wrong TRUE)

| Rule misapplied | Count | % of FP |
|----------------|-------|---------|
| Contradiction motif | 57 | 39% |
| Constant operation | 35 | 24% |
| Structural heuristic | 26 | 18% |
| Left projection | 13 | 9% |
| Right projection | 9 | 6% |
| Singleton collapse | 3 | 2% |

### False negative causes (missed TRUE)

- **"default false" fallback: 63 of 104 (61%)** — the model can't find an applicable TRUE rule, gives up and defaults to FALSE. The cheatsheet's rule coverage isn't wide enough.
- 2 cases with fabricated counterexamples (finds a "counterexample" that's actually wrong).

### Key insight

The model **over-applies cheatsheet rules** — it pattern-matches a rule name (especially contradiction motifs, constant operation, structural heuristic) without correctly verifying the preconditions. Contradiction motif alone accounts for 39% of all false positives. It also invents plausible-sounding algebraic arguments to justify wrong TRUE verdicts.

---

## GPT-OSS-20b — 372/669 correct (55.6%)

| Dataset | Accuracy | Bias |
|---------|----------|------|
| hard1 | 44/69 (63.8%) | TRUE-leaning (55% TRUE) |
| hard2 | 118/200 (59.0%) | Heavy TRUE bias (68% TRUE) |
| hard3 | 210/400 (52.5%) | Heavy TRUE bias (63% TRUE) |

### Failure breakdown (297 total)

- **65% false positives** (193) — says TRUE when answer is FALSE
- **25% false negatives** (73)
- **5% truncation** (16) — hit 16384 token limit
- **5% timeouts/errors** (15)

### False positive causes (wrong TRUE)

| Rule misapplied | Count | % of FP |
|----------------|-------|---------|
| (no RULE line — truncated reasoning) | 64 | 33% |
| Contradiction motif | 49 | 25% |
| Structural heuristic | 33 | 17% |
| Constant operation | 29 | 15% |
| Left projection | 9 | 5% |
| Right projection | 6 | 3% |

### Key insight

Two distinct failure modes:

1. **Truncation-driven errors (33% of FP):** The model starts rambling chain-of-thought reasoning, hits the 16k token limit, and the verdict is extracted from a partial, often wrong, mid-reasoning guess. Even 16384 tokens isn't enough for this model on hard problems.

2. **Same rule-misfiring as 120b** — contradiction motif, structural heuristic, and constant operation are the top offenders. The 20b model is even more aggressive in claiming "constant operation" (disjoint variable sets -> constant magma) and misapplying the structural heuristic T1/T5 rules.

---

## Comparative Summary

| Failure Mode | 120b | 20b |
|-------------|------|-----|
| Contradiction motif misfire | **39% of FP** | 25% of FP |
| Constant operation misfire | 24% of FP | 15% of FP |
| Structural heuristic misfire | 18% of FP | 17% of FP |
| Truncation (no verdict) | 2% | **10%** |
| Default-FALSE on TRUE problems | 61% of FN | 27% of FN |
| Strong TRUE bias | Moderate (55%) | **Heavy (63-68%)** |

## Actionable Takeaways

1. **Contradiction motifs C1-C15 have too-loose preconditions** — both models fire them on non-matching equations. Tightening the motif matching conditions would fix ~30-40% of false positives.
2. **Constant operation rule is over-triggered** — models claim "disjoint variable sets" when they aren't actually disjoint, or apply the conclusion incorrectly.
3. **Structural heuristic T1/T5 needs guardrails** — models aren't verifying all preconditions before firing.
4. **20b still truncates at 16k** — needs either VERDICT-first enforcement (already in prompt but ignored) or an even simpler reasoning format.
5. **Default-FALSE coverage gap** — 120b misses 61% of TRUE cases by falling through to default. The cheatsheet needs more TRUE-detection rules.
