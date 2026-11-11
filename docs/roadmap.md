# Roadmap

Eighteen levels in six groups. Each level has a learn half and a build half.

## 1 The baseline

| Level | Learn | Build |
|---|---|---|
| 1 | What a baseline is | Freeze the config and the question set, with a hash, as the baseline |
| 2 | The test set | Questions with known answers, including some with no answer, a dev and held-out split, and the correct chunk labelled per question |
| 3 | Run once, record everything | Run the evaluation once and save a per-question record |

## 2 The pipeline

| Level | Learn | Build |
|---|---|---|
| 4 | The six stages | Mark the stage boundaries in the code (find, rerank, generate) |
| 5 | Build once versus every question | Time ingest separately from query time |

## 3 Finding the broken stage

| Level | Learn | Build |
|---|---|---|
| 6 | Change one thing at a time | Swap one setting and log it in the same results row |
| 7 | Test retrieval alone | A script for recall@k and MRR with no judge |
| 8 | Retrieval fault or generation fault | Split wrong answers into retrieval misses, generation misses and correct refusals |
| 9 | Reading scores in causal order | A one-page run report that names the weakest stage |
| 10 | Choosing the setting to change | Pick one setting in that stage and run it |

## 4 Trusting the measurement

| Level | Learn | Build |
|---|---|---|
| 11 | The judge and humans | A human-labelled sample of at least 50 and agreement (kappa) per metric |
| 12 | Repeatability and noise | A small stability check: rerun, reload the model, reword the input |
| 13 | Decision rule and the held-out set | One target metric plus guardrails, and a single run on the held-out set |

## 5 Speed

| Level | Learn | Build |
|---|---|---|
| 14 | What to time | Per-stage timers written into the per-question record |
| 15 | Timing fairly | A warm-up run, then median and slowest 5 percent |
| 16 | Quality and speed together | A speed column in the results table and a latency guardrail |

## 6 Lead-engineer level

| Level | Learn | Build |
|---|---|---|
| 17 | Explain it in two minutes | A one-page walkthrough with one picture per stage |
| 18 | Run it end to end | A full pass with real numbers and a final report |
