# Whisper-for-Meeting-on-Colab

這個 Notebook 工程提供「下載影音 → 音訊降噪 → faster-whisper 轉錄 → OpenCC 正規化 → GPT-OSS-20B 摘要」的一站式流程，已針對 Colab T4 環境（15GB VRAM / 12.7GB RAM）進行調校。

## Notebook 安全機制
- 在 Utility 區塊新增了 `Notebook 語法自檢` cell，會將所有 code cell 串接後以 `compile()` 驗證語法並輸出對應行號。建議在每次修改後先執行此 cell，提前阻斷未閉合字串等錯誤。

## 非 Colab 靜態檢查流程
在本地或 CI 環境中可透過 `nbqa` 對 `.ipynb` 進行語法與格式檢查：

```bash
pip install nbqa ruff black
nbqa ruff --select=E9,F63,F7,F82 .
nbqa black --check .
```

上述指令會攔截語法錯誤（E9 類）與常見的 NameError / 控制流問題（F63/F7/F82），並保留 Black 的格式一致性。

## 參考資料
- 轉錄：faster-whisper
- 摘要：GPT-OSS-20B（llama.cpp / GGUF）
- 後處理：OpenCC、Harmony chat formatter

詳細調整與量測指標請參考 [`CHANGELOG_MEEETING_SUMMARY.md`](CHANGELOG_MEEETING_SUMMARY.md)。
