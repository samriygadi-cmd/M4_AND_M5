# Berth Rescheduling — Rule-Based (FCFS)

Notebook: `M4_Berth_Scheduling_Rule_Based.ipynb`

This notebook detects berth scheduling conflicts in the 2025 Port of Barcelona port-call data and resolves them with a first-come-first-served (FCFS) rule. It is plain Python. No LLM or API key is used.

## Requirements
- Google Colab or a local Python 3 environment
- `pandas` 2.0 or later (`format='mixed'` date parsing)
- Dataset: `escales_arribades_2025 3.csv`

## How to run
1. Open the notebook in Google Colab.
2. Choose **Runtime → Run all**.
3. Upload the CSV file when asked. A file with a different name is also accepted.

The whole notebook runs in a few seconds.

## Notebook structure
| Section | Content |
|---|---|
| Setup | Package install and imports |
| 1. Data | Load, clean and summarise the dataset |
| 2. Rescheduling | FCFS rescheduling function and run |
| 3. Evaluation | Conflict detection and constraint validation |
| 4. Success Criteria | The ten success criteria from the evaluation document |
| 5. Test Cases | Five normal and nine abnormal test cases |
| 6. Metrics | Final project metrics |

## Data cleaning
Columns used: `ESCALANUM` (vessel ID), `MOLLCODI` (berth), `MOLLMODULS` (berth modules), `ETA`, `ETD`.

A record is removed if it:
- has a missing vessel ID, berth, modules, ETA or ETD
- has invalid module text
- has ETD not after ETA
- is an exact duplicate (one copy is kept)
- has a vessel ID that still appears more than once

Result: 10,309 rows → 8,675 port calls across 36 berths. 1,449 rows had missing data and 185 were exact duplicates.

## Conflict definition
Two port calls conflict when all of these hold:
1. They are at the same berth.
2. They share at least one berth module (ranges such as `3-7` are expanded).
3. Their time intervals overlap: `ETA_A < ETD_B` and `ETA_B < ETD_A`.

Touching times, where one vessel leaves exactly when the next arrives, are not a conflict.

## Rescheduling method
For each berth, vessels are processed in order of original ETA. Each vessel starts at the earliest time, not before its original ETA, when none of its modules is occupied by a vessel already scheduled. Berth, modules and service duration never change. Vessels without a conflict keep their original times.

## Constraints checked
- Berth and modules unchanged
- ETA not earlier than the original ETA
- Service duration unchanged (ETA and ETD shifted by the same amount)
- ETD after ETA
- Every changed vessel exists and appears only once

A proposal that breaks any constraint is rejected and not applied.

## Results
| Metric | Value |
|---|---|
| Initial conflicts | 47 (87 port calls, 15 berths) |
| Detection precision / recall | 1.00 / 1.00 |
| Remaining conflicts | 0 |
| New conflicts | 0 |
| Conflict Resolution Rate (CRR) | 100% |
| Moved vessels | 58 |
| Average Added Delay (AAD) | 1,425.97 min |
| Total added delay | 82,706 min |
| Schedule Deviation | 9.53 min |
| Execution time | about 5 s |
| Rescheduling attempts | 1 |
| LLM tokens / cost | 0 / $0 |

Success criteria: 10/10 passed. Test cases: 14/14 passed.

More vessels are moved (58) than are directly involved in conflicts, because delaying one vessel can push the next vessel at the same berth.

## Metric definitions
- **CRR**: (initial conflicts − remaining conflicts) / initial conflicts × 100
- **AAD**: mean ETA delay, in minutes, of vessels that were moved
- **Remaining conflicts**: conflicting pairs in the final timetable
- **Schedule Deviation**: mean absolute ETA change, in minutes, over all 8,675 port calls
- **Execution time**: time taken by the rescheduling step
- **Rescheduling attempts**: number of rescheduling runs (1)
- **Token / LLM cost**: 0, no LLM is used

## Limitations
- FCFS always delays the later-arriving vessel. Every delayed vessel starts as soon as its module is free, but the total delay is not guaranteed to be the lowest possible for the whole timetable.
- Only time shifts are considered. Berth reassignment, tides, tugs, pilots, cargo handling and weather are out of scope.
- The detected conflicts are potential scheduling conflicts in the planned data, not confirmed operational conflicts.
