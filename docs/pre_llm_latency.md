# Pre-generation latency (Nano)

**English** | [繁體中文](pre_llm_latency.zh-TW.md)

Measured on 2026-11-12. Scope: question embedding, retrieval and context build. **The generation (LLM) stage is not included.**

## Setup

| Item | Value |
|---|---|
| Machine | Jetson Orin Nano (8 GB, CPU and GPU share memory) |
| Embedding model | ruri-v3-130m |
| Vector store | Exact search (`NumpyStore`), 641 frozen memories |
| Top-k | 5 |
| Questions | 201 (dev set and test set), each repeated 3 times; the first 5 questions are warm-up and not counted |
| Filter | Same as the question-answering flow (`MemRAG.pipeline_filter`: the person plus the question's date; if there is no event date, the record date is used; no category) |
| Context budget | 1,200 characters (`configs/pipeline.yaml`) |
| Measurement script | `profile_pre_llm.py` (in `scripts/` of the RAG-TEST project; not included in this repository) |
| Nano state during measurement | Idle (load 0.43, about 5.8 GB memory available, llama-server not running); **resident services were not paused with `nano_measure.sh`** |

The three stages are split the same way as `ask` in `application/answer.py`:

1. **Question embedding**: add the query prefix and encode (`embed_queries`).
2. **Retrieval**: filter by `person_id` and date, take the top-k, and fetch the full memories (`store.search` plus `store.get`).
3. **Context build**: remove duplicates, sort by score, and truncate to the character budget (`build_context`).

## Results

### GPU (CUDA)

| Stage | p50 (ms) | p95 (ms) | p99 (ms) | Share |
|---|---|---|---|---|
| 1. Question embedding | 31.31 | 32.49 | 34.86 | 99.0% |
| 2. Retrieval | 0.28 | 0.37 | 0.46 | 0.9% |
| 3. Context build | 0.04 | 0.05 | 0.06 | 0.1% |
| **Total** | **31.64** | **32.83** | **35.38** | 100% |

### CPU (comparison)

| Stage | p50 (ms) | p95 (ms) | p99 (ms) | Share |
|---|---|---|---|---|
| 1. Question embedding | 93.07 | 99.67 | 105.17 | 99.4% |
| 2. Retrieval | 0.51 | 0.57 | 0.66 | 0.5% |
| 3. Context build | 0.07 | 0.07 | 0.10 | 0.1% |
| **Total** | **93.66** | **100.25** | **105.72** | 100% |

Model load time (first load, not included in the per-question times): about 46.8 s on GPU, about 10.2 s on CPU.

## Conclusions

- The three pre-generation stages total about 32 ms on GPU and about 94 ms on CPU.
- **The bottleneck is question embedding**, which takes more than 99%. Retrieval and context build together take less than 1 ms.
- GPU is about 3 times faster than CPU, mostly because of embedding.
- Consistent with the 2026-11-09 retrieval measurement (`reports/records/2026-11-09_retrieval-profile.json`): GPU 31.6 ms, CPU 94.8 ms (exact search, top-k 5).
- With the LLM answer added (p50 2.6 s, see `reports/records/2026-11-09_answer-pipeline.json`), one question takes about 2.7 s, and the three pre-generation stages are only about 1% of it.

## Limitations

- Measured once, with resident services not paused. Before this goes into a formal report, run it again with the resident services paused.
- The Nano is a development and test platform, not the production hardware (D2). Production numbers must be re-measured on the deployment server.
- There are only 641 memories. For retrieval cost at larger data sizes, see experiment B in 5.1.7 (exact search over 50k memories takes about 0.29 ms).
- The code that ran on the Nano is from the RAG-TEST project (commit `f670dd1`), with only the measurement script copied over.

## How to reproduce

Run from the root of the RAG-TEST project (on the Nano):

```bash
uv run python scripts/profile_pre_llm.py --model ruri-v3-130m --device cuda --out pre_llm_cuda.json
uv run python scripts/profile_pre_llm.py --model ruri-v3-130m --device cpu  --out pre_llm_cpu.json
```

`--fake` runs a smoke test on a machine without the model (the numbers are meaningless). `--out` writes the full summary JSON.
