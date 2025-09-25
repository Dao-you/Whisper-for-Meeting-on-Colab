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

## 常見錯誤對照表
| 錯誤訊息 | 可能原因 | 快速修復 |
| --- | --- | --- |
| `SyntaxError: unterminated string literal` / `unterminated f-string literal` | 跨行 `print()` / `f-string` 未以括號安全串接，或遺漏 `f` 前綴 | 將訊息包在單一 `print()` 內並以括號自動串接字串，或拆成多個 `print()`；含插值時保留 `f` |
| `NameError: name 'importlib' is not defined` | 在 `importlib.util.find_spec` 的 cell 缺少 `import importlib` | 在同一個 Notebook cell 頂部補上 `import importlib` |

上述指令會攔截語法錯誤（E9 類）與常見的 NameError / 控制流問題（F63/F7/F82），並保留 Black 的格式一致性。

## 參考資料
- 轉錄：faster-whisper
- 摘要：GPT-OSS-20B（llama.cpp / GGUF）
- 後處理：OpenCC、Harmony chat formatter

詳細調整與量測指標請參考 [`CHANGELOG_MEEETING_SUMMARY.md`](CHANGELOG_MEEETING_SUMMARY.md)。
