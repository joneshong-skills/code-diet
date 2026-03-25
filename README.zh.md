[English](README.md) | [繁體中文](README.zh.md)

# code-diet

掃描程式碼庫中的重複、死碼與抽象機會——提出最小化的共用提取建議。

## 說明

Code Diet 對 Workshop 程式碼庫執行三個平行掃描，找出模組間幾乎相同的邏輯、未使用的函式與 import，以及模組用不同方式解決相同問題的地方。在建議任何提取至共用程式碼之前，它會套用嚴格的四問決策框架，確保只有具備兩個以上確認消費者的模式才會移入 `shared/`。輸出為存放於 `~/workshop/outputs/code-diet/` 的優先級排序瘦身報告。

## 功能特色

- 三個平行掃描：重複偵測、死碼偵測、模式分歧
- 四問「提取 vs 保留」決策框架（2+ 消費者、與 domain 無關、泛型簽名、未來復用）
- 將發現分類為 `→shared`、`✂️ dead`、`→adopt` 或 `✓ keep`
- 含工時估算的優先級報告（快速修復 < 30 分鐘、中等工作 1-3 小時、未來考量）
- 範圍控制：針對單一模組、關鍵字或完整程式碼庫
- 結果以 intelflow 情報報告儲存供趨勢追蹤
- 組合優於繼承哲學：函式，而非類別階層

## 使用方式

```
/code-diet [範圍]
```

範例：

- `/code-diet` — 掃描 `core/src/`（預設）
- `/code-diet full` — 掃描 core/ + stations/ + mcp/ + libs/
- `/code-diet memvault` — 專注於 memvault 模組 vs shared
- `/code-diet search` — 只針對搜尋相關程式碼路徑
- "整理重複程式碼"
- "找死碼"

## 運作原理

Code Diet 啟動三個平行探索 agent，分別針對重複、死碼與模式分歧。接著由規劃 agent 對每個發現套用四問框架，產生已分類與優先排序的清單。最終報告寫入 `~/workshop/outputs/code-diet/`，並以 intelflow 報告形式持久化。提案變更的執行刻意留給開發者決定——技能提議，人類裁決。

## 系統需求

- Claude Code CLI
- Workshop intelflow CLI（`~/.local/bin/python3 ~/workshop/core/cli/intelflow.py`）
- 存取 `core/src/` 及選用的 `stations/`、`mcp/`、`libs/`

## 授權條款

MIT
