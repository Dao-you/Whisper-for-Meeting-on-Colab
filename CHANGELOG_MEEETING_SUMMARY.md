# Whisper Meeting Pipeline – Changelog & Notes

## 本次調整摘要
- ✅ 修復 `run_transcription_attempt()` 內未閉合的 f-string，改用多行 `print()` 並統一串接訊息，避免再度觸發 `SyntaxError`。
- ✅ 僅保留單一 `ensure_harmony_formatter()` 實作（上方 Utility 區塊），刪除後段重複定義以降低維護風險。
- ✅ 移除未使用的 `loop_buffer`，改以新的去重邏輯與統計欄位追蹤語音片段。
- ✅ De-loop sentinel 增設升級門檻：需要「連續 ≥8 次且 ≥10 秒」或「同一分鐘內 ≥2 件」才會升級，並於 log 中列出觸發理由。
- ✅ 轉錄新增 `raw_segment_seconds / vad_removed_seconds / post_filter_removed_seconds`，精確拆分 VAD 與後處理的貢獻；日誌同步展示。
- ✅ Harmony stream fallback 增加最輕度清理，僅移除孤立 `<|...|>` 標記，避免破損模板寫入 Markdown。
- ✅ Notebook 前段加入 `Notebook 語法自檢` cell（`compile()` 驗證），README 補充 `nbqa ruff` / `nbqa black` 靜態檢查流程。
- ✅ CHANGELOG 補充 ctx window 的回退策略：若 12k/16k 載入失敗或品質異常，可退回 8k~12k 並檢查 RoPE / 長上下文設定。

## 預設參數（Developer Options 擴充）
| 參數 | 預設 | 備註 |
| --- | --- | --- |
| `SELECTED_VAD_SILENCE_PRESET` | `"Aggressive"` | Aggressive=0.8s (800ms)，Conservative=1.2s (1200ms) |
| `CTX_WINDOW_CANDIDATES` | `[12288, 16384, 8192]` | 若 12k/16k 載入失敗或品質異常，優先回退 8k~12k；必要時檢查模型的 RoPE/長上下文設定 |
| `STREAM_FALLBACK_USED` | `False` | 執行時自動覆寫，摘要完成後會在 metrics 中顯示 |

> ⚙️ 開發者可直接在 Notebook 內修改上述常數；所有參數旁均附有中英註解，便於 A/B 測試。

## Notebook / CI 檢查補充
- Notebook 內的 `Notebook 語法自檢` cell 會將所有 code cell 合併後以 `compile()` 驗證，失敗時輸出對應行號與上下文。
- 非 Colab 環境建議執行：
  - `pip install nbqa ruff black`
  - `nbqa ruff --select=E9,F63,F7,F82 .`
  - `nbqa black --check .`

## 觀測指標輸出
- 轉錄階段會在 `print("[Transcription Metrics]")` 下方列出：
  - compute_type / VAD preset / 語言偵測 / 音訊秒數與 raw speech / VAD 過濾秒數 / 後處理過濾秒數 / 最終保留秒數
  - compression ratio 觸發次數、最大重複次數、各次重複事件（含時間區間、持續秒數、採用 guardrail），以及升級理由（若觸發）
- 摘要階段會在 `print("[Summary Metrics]")` 下列出：
  - ctx window 候選與最終選擇
  - 每個 map 分段的輸入/輸出 tokens 與是否命中 `map_max_new_tokens`
  - reduce 階段的輸入/輸出 tokens 與是否命中 `reduce_max_new_tokens`
  - 若使用 fallback 解析器，會額外印出 `stream-flush fallback used`

## 範例輸出
- `sample_run/sample_summary.md`：以虛構會議內容示範最終 Markdown 結構（整體提要 / 章節要點 / 可執行重點）。
- `sample_run/sample_output.txt`：節錄終端日誌，展示轉錄與摘要指標的實際排版（含 de-loop 升級理由）。

> 📌 範例檔案僅示意格式，實際專案請以真實轉錄結果覆蓋。

## 版本資訊（執行時請補充）
| 套件 | 版本建議 | 取得方式 |
| --- | --- | --- |
| `faster-whisper` | *(於 Colab 執行後以 `pip show faster-whisper` 確認)* | `pip install --upgrade faster-whisper` |
| `llama-cpp-python` | 與 GPU CUDA tag 相容版本（例：`cu124` / `cu125`） | Notebook 已自動偵測並載入對應 wheel |
| GGUF 模型 | `unsloth/gpt-oss-20b-Q4_K_M.gguf` | `huggingface_hub.snapshot_download` |
| Colab GPU | T4（15GB VRAM） | `nvidia-smi` 查詢 |

> 建議在每次正式跑完流程後，於此表更新實際版本號與 GPU 資訊，方便回溯。
