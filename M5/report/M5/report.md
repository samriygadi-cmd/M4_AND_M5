# M5: evaluation harness and baseline scores

## Setup
- Golden set: 50 cases, SHA-256 `fd342bbebf934304…`
- Seeds: [11, 23, 47]. The seed sets the row order of the table given to the system; all conditions get the same order.
- Model requested: `deepseek-chat`; model reported by the API: `deepseek-flash`
- Temperature 0.2, at most 3 rescheduling attempts, tools: none
- Prices: 0.28 / 0.42 USD per 1M input / output tokens (cache discounts ignored)
- Last run: 2026-11-02 18:01:57

Cells show mean ± sample SD across seeds [min–max]. With three seeds, the SD is a rough spread, not a precise estimate.

## Golden set
| Tier | Cases | Vessels | Conflict pairs |
|---|---|---|---|
| easy | 16 | 48 | 16 |
| hard | 11 | 162 | 23 |
| medium | 13 | 142 | 13 |
| negative | 8 | 76 | 0 |
| stress | 2 | 20 | 54 |
| **All** | **50** | 448 | 106 |

## Baseline scores with variance
| Metric | Do-nothing | Non-agent (FCFS) | Single agent |
|---|---|---|---|
| Solved cases: valid and conflict-free (%) | 16.0 ± 0.0 [16.0–16.0] | 100.0 ± 0.0 [100.0–100.0] | 36.7 ± 4.2 [32.0–40.0] |
| Solved, cases with conflicts (%) | 0.0 ± 0.0 [0.0–0.0] | 100.0 ± 0.0 [100.0–100.0] | 34.9 ± 3.6 [31.0–38.1] |
| Solved, cases without conflicts (%) | 100.0 ± 0.0 [100.0–100.0] | 100.0 ± 0.0 [100.0–100.0] | 45.8 ± 7.2 [37.5–50.0] |
| CRR, schedule as proposed (%) | 0.0 ± 0.0 [0.0–0.0] | 100.0 ± 0.0 [100.0–100.0] | 48.4 ± 16.5 [30.2–62.3] |
| CRR, invalid proposals rejected (%) | 0.0 ± 0.0 [0.0–0.0] | 100.0 ± 0.0 [100.0–100.0] | 32.4 ± 17.3 [18.9–51.9] |
| CRR, as proposed, real cases only (%) | 0.0 ± 0.0 [0.0–0.0] | 100.0 ± 0.0 [100.0–100.0] | 5.8 ± 19.5 [-15.4–23.1] |
| CRR, invalid rejected, real cases only (%) | 0.0 ± 0.0 [0.0–0.0] | 100.0 ± 0.0 [100.0–100.0] | -1.3 ± 4.0 [-5.8–1.9] |
| Remaining conflicts (total) (pairs) | 106.0 ± 0.0 [106.0–106.0] | 0.0 ± 0.0 [0.0–0.0] | 54.7 ± 17.5 [40.0–74.0] |
| Remaining conflicts, real cases only (pairs) | 52.0 ± 0.0 [52.0–52.0] | 0.0 ± 0.0 [0.0–0.0] | 49.0 ± 10.1 [40.0–60.0] |
| New conflicts (total) (pairs) | 0.0 ± 0.0 [0.0–0.0] | 0.0 ± 0.0 [0.0–0.0] | 36.0 ± 9.5 [27.0–46.0] |
| Runs with no constraint violation (%) | 100.0 ± 0.0 [100.0–100.0] | 100.0 ± 0.0 [100.0–100.0] | 55.3 ± 3.1 [52.0–58.0] |
| Average added delay (AAD) (min) | n/a | 1,317.8 ± 0.0 [1,317.8–1,317.8] | 2,110.3 ± 500.3 [1,665.1–2,651.8] |
| Total added delay (h) | 0.0 ± 0.0 [0.0–0.0] | 1,625.2 ± 0.0 [1,625.2–1,625.2] | 7,039.0 ± 1,618.6 [5,855.6–8,883.5] |
| Schedule deviation (min) | 0.0 ± 0.0 [0.0–0.0] | 217.7 ± 0.0 [217.7–217.7] | 1,062.5 ± 152.2 [958.5–1,237.1] |
| Total delay / total optimal delay, solved cases (ratio) | n/a | 1.32 ± 0.00 [1.32–1.32] | 1.92 ± 0.77 [1.15–2.68] |
| Median per-case delay / optimal, solved cases (ratio) | n/a | 1.00 ± 0.00 [1.00–1.00] | 1.00 ± 0.00 [1.00–1.00] |
| Detection precision | n/a | 1.000 ± 0.000 [1.000–1.000] | 0.243 ± 0.014 [0.230–0.258] |
| Detection recall | 0.000 ± 0.000 [0.000–0.000] | 1.000 ± 0.000 [1.000–1.000] | 0.852 ± 0.054 [0.792–0.896] |
| Detection F1 | 0.000 ± 0.000 [0.000–0.000] | 1.000 ± 0.000 [1.000–1.000] | 0.378 ± 0.020 [0.357–0.397] |
| Success criteria passed (of 10) | 6.64 ± 0.00 [6.64–6.64] | 10.00 ± 0.00 [10.00–10.00] | 7.07 ± 0.12 [6.94–7.18] |

## Trajectory-level metrics
| Metric | Do-nothing | Non-agent (FCFS) | Single agent |
|---|---|---|---|
| Rescheduling attempts per case | 1.00 ± 0.00 [1.00–1.00] | 1.00 ± 0.00 [1.00–1.00] | 2.23 ± 0.05 [2.18–2.28] |
| LLM calls per case | 0.00 ± 0.00 [0.00–0.00] | 0.00 ± 0.00 [0.00–0.00] | 2.23 ± 0.05 [2.18–2.28] |
| Attempts until first solved proposal | 1.00 ± 0.00 [1.00–1.00] | 1.00 ± 0.00 [1.00–1.00] | 1.04 ± 0.06 [1.00–1.11] |
| False termination (claimed done, not solved) (%) | 84.0 ± 0.0 [84.0–84.0] | 0.0 ± 0.0 [0.0–0.0] | 44.0 ± 2.0 [42.0–46.0] |
| Invalid replies retried per case | 0.000 ± 0.000 [0.000–0.000] | 0.000 ± 0.000 [0.000–0.000] | 0.007 ± 0.012 [0.000–0.020] |

## Cost and latency per condition
| Metric | Do-nothing | Non-agent (FCFS) | Single agent |
|---|---|---|---|
| Latency per case (s) | 0.0000 ± 0.0000 [0.0000–0.0000] | 0.0000 ± 0.0000 [0.0000–0.0000] | 5.4877 ± 0.2159 [5.2405–5.6389] |
| Tokens per case | 0 ± 0 [0–0] | 0 ± 0 [0–0] | 4,429 ± 195 [4,205–4,562] |
| Cost per case (USD) | 0.00000 ± 0.00000 [0.00000–0.00000] | 0.00000 ± 0.00000 [0.00000–0.00000] | 0.00142 ± 0.00006 [0.00135–0.00146] |
| Cost of one full pass of the golden set (USD) | 0.0000 ± 0.0000 [0.0000–0.0000] | 0.0000 ± 0.0000 [0.0000–0.0000] | 0.0708 ± 0.0031 [0.0673–0.0731] |
| Solved cases per 1K tokens | n/a | n/a | 0.083 ± 0.013 [0.070–0.095] |

## Conflicts by attempt
| Condition | Attempt | Solved (%) | Remaining conflicts |
|---|---|---|---|
| Single agent | 1 | 37.3 | 55.7 |
| Single agent | 2 | 36.7 | 57.7 |
| Single agent | 3 | 36.7 | 54.7 |

## Failure analysis
### By tier
| Tier (solved %) | Do-nothing | Non-agent (FCFS) | Single agent |
|---|---|---|---|
| easy | 0.0 ± 0.0 | 100.0 ± 0.0 | 77.1 ± 3.6 |
| hard | 0.0 ± 0.0 | 100.0 ± 0.0 | 0.0 ± 0.0 |
| medium | 0.0 ± 0.0 | 100.0 ± 0.0 | 10.3 ± 4.4 |
| negative | 100.0 ± 0.0 | 100.0 ± 0.0 | 45.8 ± 7.2 |
| stress | 0.0 ± 0.0 | 100.0 ± 0.0 | 50.0 ± 50.0 |

### By failure label
A run can carry several labels.

| Failure label (share of runs, %) | Do-nothing | Non-agent (FCFS) | Single agent |
|---|---|---|---|
| above_optimal_delay | 0.0 | 14.0 | 8.0 |
| false_alarm | 0.0 | 0.0 | 66.7 |
| false_termination | 84.0 | 0.0 | 44.0 |
| invalid_duration | 0.0 | 0.0 | 12.7 |
| invalid_earlier_eta | 0.0 | 0.0 | 36.7 |
| missed_conflict | 84.0 | 0.0 | 20.0 |
| moved_without_conflict | 0.0 | 0.0 | 12.0 |
| new_conflict | 0.0 | 0.0 | 26.0 |
| unresolved_original | 84.0 | 0.0 | 22.0 |

### Do the cases separate the systems?
- failed by every LLM condition in every seed: 23 cases
- separates conditions or seeds: 16 cases
- solved by every LLM condition in every seed: 11 cases
- Cases no LLM condition solved: N01, N03, R01, R12, R16, R17, R20, R21, R22, R24, R25, R26, R27, R28, R29, R30, R31, R32, R33, R34, R35, R36, R37
