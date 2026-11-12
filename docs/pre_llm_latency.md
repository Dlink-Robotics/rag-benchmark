# 生成前三階段耗時（Nano）

量測日期：2026-11-12。範圍：問題 Embedding、檢索、Context build，**不含生成（LLM）階段**。

## 設定

| 項目 | 值 |
|---|---|
| 機器 | Jetson Orin Nano（8 GB，CPU 與 GPU 共用） |
| Embedding 模型 | ruri-v3-130m |
| 向量庫 | 精確搜尋（`NumpyStore`），641 筆凍結記憶 |
| Top-k | 5 |
| 題目 | 201 題（開發集＋測試集），每題重複 3 次，前 5 題暖機不計 |
| 過濾 | 與問答流程相同（`MemRAG.pipeline_filter`：本人＋題目的日期，沒有事件日期改看記錄日，不用類別） |
| Context 預算 | 1,200 字（`configs/pipeline.yaml`） |
| 量測腳本 | `profile_pre_llm.py`（在 RAG-TEST 專案的 `scripts/`，本儲存庫沒有收錄） |
| 量測時的 Nano 狀態 | 閒置（load 0.43，可用記憶體約 5.8 GB，llama-server 未執行）；**未使用 `nano_measure.sh` 暫停常駐服務** |

三段的切法與 `application/answer.py` 的 `ask` 相同：

1. **問題 Embedding**：加 query prefix 並編碼（`embed_queries`）。
2. **檢索**：依 `person_id`／日期過濾後取 Top-k，並取回完整記憶（`store.search` 加 `store.get`）。
3. **Context build**：去重、依分數排序、依字數預算截斷（`build_context`）。

## 結果

### GPU（CUDA）

| 階段 | p50 (ms) | p95 (ms) | p99 (ms) | 佔比 |
|---|---|---|---|---|
| 1. 問題 Embedding | 31.31 | 32.49 | 34.86 | 99.0% |
| 2. 檢索 | 0.28 | 0.37 | 0.46 | 0.9% |
| 3. Context build | 0.04 | 0.05 | 0.06 | 0.1% |
| **合計** | **31.64** | **32.83** | **35.38** | 100% |

### CPU（對照）

| 階段 | p50 (ms) | p95 (ms) | p99 (ms) | 佔比 |
|---|---|---|---|---|
| 1. 問題 Embedding | 93.07 | 99.67 | 105.17 | 99.4% |
| 2. 檢索 | 0.51 | 0.57 | 0.66 | 0.5% |
| 3. Context build | 0.07 | 0.07 | 0.10 | 0.1% |
| **合計** | **93.66** | **100.25** | **105.72** | 100% |

模型載入（第一次，不含在每題耗時內）：GPU 約 46.8 秒，CPU 約 10.2 秒。

## 結論

- 生成前三段合計：GPU 約 32 ms，CPU 約 94 ms。
- **瓶頸是問題 Embedding**，佔 99% 以上；檢索與 Context build 加起來不到 1 ms。
- GPU 比 CPU 快約 3 倍，主要差在 Embedding。
- 與 2026-11-09 的檢索量測（`reports/records/2026-11-09_retrieval-profile.json`）一致：GPU 31.6 ms、CPU 94.8 ms（精確搜尋、Top-k 5）。
- 加上 LLM 回答（p50 2.6 秒，見 `reports/records/2026-11-09_answer-pipeline.json`）後，整題約 2.7 秒，生成前三段只佔約 1%。

## 限制

- 只量一次，未暫停常駐服務；要寫進正式報告，建議在暫停常駐服務的情況下再跑一次。
- Nano 是開發與測試平台，不是正式部署硬體（D2）；正式數字要在部署伺服器重測。
- 記憶只有 641 筆；資料量變大後的檢索成本見 5.1.7 實驗 B（精確搜尋 50k 筆約 0.29 ms）。
- Nano 上跑的是 RAG-TEST 專案的程式碼（commit `f670dd1`），只額外複製了量測腳本。

## 重現方式

在 RAG-TEST 專案根目錄（Nano 上）執行：

```bash
uv run python scripts/profile_pre_llm.py --model ruri-v3-130m --device cuda --out pre_llm_cuda.json
uv run python scripts/profile_pre_llm.py --model ruri-v3-130m --device cpu  --out pre_llm_cpu.json
```

`--fake` 可在沒有模型的機器上做冒煙測試（數字沒有意義）。`--out` 會寫出完整摘要 JSON。
