# Single-Agent Berth Rescheduling — Baseline

A **single LLM agent** baseline for detecting and resolving berth scheduling conflicts in the Port of Barcelona 2025 vessel calls (`escales_arribades_2025`).
The agent reads the schedule, finds conflicting vessel pairs by itself, and proposes a delayed timetable.
Python never tells the agent which vessels conflict or how to move them. Python only prepares the data, and **checks the result independently afterwards** against 10 success criteria.

Notebook: `M4_Berth_Scheduling_Baseline_Single_Agent.ipynb`

```
1. Data preparation  →  2. LLM agent (detect + resolve)  →  3. Independent validation
```

---

## Problem definition

Two port calls are **in conflict** when all three conditions hold:

| Condition | Rule |
|---|---|
| Same berth | `berth_A == berth_B` |
| Shared quay module | the module sets intersect (`3-6` = {3,4,5,6}; `1,4` = {1,4}) |
| Overlapping stay | `eta_A < etd_B` and `eta_B < etd_A` |

Touching stays (`etd_A == eta_B`) are **not** conflicts. Several vessels can be at the same berth at the same time if they use different modules.

The agent may only **delay** vessels: berth, modules and service duration stay the same, and the new ETA can never be earlier than the original one.

---

## Requirements

- Google Colab (recommended) or Jupyter with Python 3.10+
- `openai`, `pandas` (installed by the first code cell)
- A **DeepSeek API key** (any OpenAI-compatible endpoint works, see *Configuration*)
- The input CSV `escales_arribades_2025 3.csv` with at least these columns:

| CSV column | Meaning | Internal name |
|---|---|---|
| `ESCALANUM` | port-call ID | `vessel_id` |
| `MOLLCODI` | berth code | `berth` |
| `MOLLMODULS` | quay modules, e.g. `3-6`, `1,4` | `modules` |
| `ETA` | estimated time of arrival | `eta` |
| `ETD` | estimated time of departure | `etd` |

## How to run

1. Open the notebook in Colab.
2. Add your key under **Colab → 🔑 Secrets** as `DEEPSEEK_API_KEY` and enable notebook access.
   Outside Colab, set the environment variable `DEEPSEEK_API_KEY` instead.
3. **Runtime → Restart and run all**. If the CSV is not in the working folder, the notebook asks you to upload it.
4. Read the **scorecard** (10 criteria) and the **metrics table** in section 3.
5. Optional: set `SAVE_RESULTS = True` in the last cell to export the results (see *Outputs*).

Always use *Restart and run all*. Running single cells in a different order can mix variables from different runs.

---

## Notebook structure

### 1. Data preparation
- Loads the CSV and skips broken lines (`on_bad_lines='skip'`).
- Cleaning: drops rows with missing values, `ETD ≤ ETA` or invalid module codes, then removes exact duplicates and **all** rows whose vessel ID appears more than once. Times are rounded down to the minute.
- `find_conflicts()` is the reference conflict detector. Its behaviour is checked by three built-in unit tests (overlapping modules, touching stays, different berths).
- Last run: 10,309 raw rows → **8,675 cleaned rows**, 36 berths, **47 reference conflicts**.

### Batching
- Only berths that contain at least one overlap are sent to the agent (15 berths, 5,817 rows). The agent is **not** told which vessels conflict.
- Each berth is split into batches of **25 vessels** (`BATCH_ROWS`), sorted by ETA. The batches of one berth are processed one after another, and different berths run in parallel.
- Vessels from earlier batches that are still at the quay are added as **`fixed=1` context rows**, using the times the agent already gave them. This way no conflict is split across two batches. The agent must not move these rows.
- Modules are written out in full (`3-6` → `3,4,5,6`).

### 2. Single LLM agent
- One system prompt (`PROMPT`) defines the conflict rule, the two tasks (detect, resolve) and the hard constraints.
- The agent replies with JSON:
  ```json
  {"conflicts": [{"a": "ID_A", "b": "ID_B", "shared_modules": [3, 4]}],
   "changes":   [{"vessel_id": "ID", "new_eta": "YYYY-MM-DD HH:MM", "new_etd": "YYYY-MM-DD HH:MM"}],
   "notes": "one short sentence"}
  ```
- Invalid or failed replies are retried up to `MAX_ATTEMPTS` times. A batch that still fails is reported as failed and is **never** filled in by Python.

### 3. Independent validation
The agent's changes are applied **exactly as proposed**, with no repairs. The resulting timetable is checked against all criteria:

| # | Success criterion | How it is checked |
|---|---|---|
| 1 | Detect conflicts correctly | reported pairs vs. reference pairs → precision / recall / F1 (pass = F1 1.0) |
| 2 | Resolve conflicts | no original conflict remains |
| 3 | Minimize added delay | every moved vessel is *tight* (starts exactly when a blocking vessel leaves); total delay is also compared with a greedy first-come-first-served reference |
| 4 | Valid rescheduled timetable | all batches answered, known and unique IDs, parseable dates |
| 5 | ETA before ETD | every row of the final timetable |
| 6 | Berth & modules preserved | final vs. original |
| 7 | Service duration preserved | `new_etd − new_eta == etd − eta` |
| 8 | No new conflicts | conflicts in the final timetable that were not in the original = ∅ |
| 9 | ETA not earlier than original | every row of the final timetable |
| 10 | Final validation | criteria 2 and 4–9 all pass |

---

## Metrics

| Metric | Meaning |
|---|---|
| Initial conflicts | reference conflict pairs in the cleaned data |
| TP / FP / FN, precision / recall / F1 | detection quality of the agent |
| Remaining original conflicts | original pairs still present after rescheduling |
| New conflicts | pairs created by the agent's changes |
| CRR | `(initial − all conflicts in final) / initial × 100`. Can be negative when the agent creates more conflicts than it resolves |
| Moved vessels, total / average (AAD) / max added delay | cost of the rescheduling |
| Greedy FCFS reference delay | delay of a simple first-come-first-served schedule, used as a yardstick |
| Constraint violations | number of broken hard constraints |
| Batches / failed / LLM calls, execution time, tokens, estimated cost | efficiency |

Also shown: the rescheduled vessels, a **per-berth detection table** (real / reported / found / false alarms), the conflicts left in the final timetable, and the list of violations.

---

## Configuration

All settings are at the top of their cells:

| Setting | Default | Notes |
|---|---|---|
| `MODEL` | `deepseek-chat` | `deepseek-reasoner` is slower and more expensive, but much more careful |
| `BASE_URL` | `https://api.deepseek.com` | any OpenAI-compatible endpoint |
| `BATCH_ROWS` | `25` | vessels per batch (context rows come on top) |
| `MAX_ATTEMPTS` | `3` | retries per batch |
| `MAX_WORKERS` | `16` | berths processed in parallel. Keep it **at or below your API concurrency limit** (DeepSeek allowed 5–8 in our tests), otherwise you get `429 Too many requests` |
| `PRICE_IN`, `PRICE_OUT` | `0.28`, `0.42` | USD per 1M tokens. Check the provider's current prices |
| `SAVE_RESULTS` | `False` | export results to files |

## Outputs (when `SAVE_RESULTS = True`)

| File | Content |
|---|---|
| `rescheduled_timetable_<stamp>.csv` | the full timetable after the agent's changes |
| `baseline_metrics_<stamp>.csv` | scorecard and metrics table |
| `agent_replies_<stamp>.json` | raw reply, error, attempts and tokens for every batch |

---

## Results (reference run, `deepseek-chat`, 2026-10-14)

| | Value |
|---|---|
| Score | 2 / 10 criteria passed (5 ETA before ETD, 6 berth & modules preserved) |
| Detection | 30 / 47 found, 2,617 false alarms (P 0.01, R 0.64, F1 0.02) |
| Original conflicts resolved | 34 / 47 |
| New conflicts | 452 |
| Moved vessels / added delay | 1,905 / 68,093 h (greedy FCFS reference: 1,378 h) |
| Constraint violations | 264 (250 earlier ETAs, 13 changed durations, 1 other) |
| Runtime / cost | 315 s, 404k input + 198k output tokens, ≈ 0.20 USD |

**Interpretation.** On busy berths, many vessels are at the quay at the same time on different modules. The single agent treats many of these as conflicts, so it reports thousands of false alarms. It then moves vessels that were never in conflict, which creates real new conflicts in later batches. Its date arithmetic is also unreliable: some ETAs move earlier and some durations change. A few `deepseek-reasoner` batches that completed before rate limits stopped the run were correct: no false alarms, and the real conflict was resolved with the smallest possible delay. A full reasoner run has not been completed yet.

## Known limitations

- **Hard constraints are not enforced on the agent.** This is intended for a baseline: every violation is measured, never repaired.
- **Fixed context rows can cause false alarms.** A fixed row shown at its new time can overlap a vessel in the next batch. The agent may report that pair, and it is counted as a false alarm because it is not an original conflict.
- **Criterion 3 checks tightness, not optimality.** A schedule with no wasted delay on each move can still have a larger total delay than the best possible schedule.
- **Rate limits.** Too many parallel requests cause `429` errors and failed batches. Lower `MAX_WORKERS`.
- **Results vary between runs.** Results depend on the model version and provider. Record the model name and date with every result.
