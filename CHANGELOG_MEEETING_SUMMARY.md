# Whisper Meeting Pipeline – Changelog & Notes

## 本次調整摘要
- ✅ 修正語義分段輸入型別，所有轉錄片段均以 `(start, end, text)` tuple 紀錄，`build_semantic_segments()` 無須再解讀 faster-whisper 的 `Segment` 物件。
- ✅ 新增 VAD 靜默雙預設（Aggressive 0.8s / Conservative 1.2s），預設採 Aggressive，並於日誌顯示實際採用的毫秒值。
- ✅ 導入「De-loop sentinel」環（ring buffer）偵測：自動記錄同句重複事件，必要時依序切換 `condition_on_previous_text=False` 與更嚴格的 `no_repeat_ngram_size=3 / compression_ratio_threshold=1.3`。
- ✅ 動態嘗試 `ctx_window`（12288 → 16384 → 8192）載入 GPT-OSS-20B，並在成功後列印實際使用的 context window。
- ✅ `stream_harmony_final_pieces()` 增強：若只收到 assistant 或完全無 Harmony 標記，會自動 flush 緩存並列印 `stream-flush fallback used`，確保不產生空白輸出。
- ✅ 轉錄 / 摘要皆新增量測指標：
  - 轉錄：compute_type、語言與機率、音訊長度、保留語音秒數 / VAD 過濾秒數、compression ratio 觸發次數、最大重複次數與事件清單。
  - 摘要：每段 map 的 input/output tokens、是否打到 `map_max_new_tokens`、reduce 階段 token 統計與是否逼近上限、ctx window 選擇、stream fallback 狀態。

## 預設參數（Developer Options 擴充）
| 參數 | 預設 | 備註 |
| --- | --- | --- |
| `SELECTED_VAD_SILENCE_PRESET` | `"Aggressive"` | Aggressive=0.8s (800ms)，Conservative=1.2s (1200ms) |
| `CTX_WINDOW_CANDIDATES` | `[12288, 16384, 8192]` | 逐一嘗試，可自行調整順序或加入 24576 |
| `STREAM_FALLBACK_USED` | `False` | 執行時自動覆寫，摘要完成後會在 metrics 中顯示 |

> ⚙️ 開發者可直接在 Notebook 內修改上述常數；所有參數旁均附有中英註解，便於 A/B 測試。

## 觀測指標輸出
- 轉錄階段會在 `print("[Transcription Metrics]")` 下方列出：
  - compute_type / VAD preset / 語言偵測 / 音訊秒數與 VAD 過濾秒數
  - compression ratio 觸發次數、最大重複次數、各次重複事件（含時間區間與採用 guardrail）
- 摘要階段會在 `print("[Summary Metrics]")` 下列出：
  - ctx window 候選與最終選擇
  - 每個 map 分段的輸入/輸出 tokens 與是否命中 `map_max_new_tokens`
  - reduce 階段的輸入/輸出 tokens 與是否命中 `reduce_max_new_tokens`
  - 若使用 fallback 解析器，會額外印出 `stream-flush fallback used`

## 範例輸出
- `sample_run/sample_summary.md`：以虛構會議內容示範最終 Markdown 結構（整體提要 / 章節要點 / 可執行重點）。
- `sample_run/sample_output.txt`：節錄終端日誌，展示轉錄與摘要指標的實際排版。

> 📌 範例檔案僅示意格式，實際專案請以真實轉錄結果覆蓋。

## 版本資訊（執行時請補充）
| 套件 | 版本建議 | 取得方式 |
| --- | --- | --- |
| `faster-whisper` | *(於 Colab 執行後以 `pip show faster-whisper` 確認)* | `pip install --upgrade faster-whisper` |
| `llama-cpp-python` | 與 GPU CUDA tag 相容版本（例：`cu124` / `cu125`） | Notebook 已自動偵測並載入對應 wheel |
| GGUF 模型 | `unsloth/gpt-oss-20b-Q4_K_M.gguf` | `huggingface_hub.snapshot_download` |
| Colab GPU | T4（15GB VRAM） | `nvidia-smi` 查詢 |

> 建議在每次正式跑完流程後，於此表更新實際版本號與 GPU 資訊，方便回溯。

