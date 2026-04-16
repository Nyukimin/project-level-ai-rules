---
description: "Chrome DevTools MCP 安全ルール（共通）"
alwaysApply: true
---

# 35-chrome-devtools-safety - Chrome DevTools MCP 安全ルール

## 1. 使用禁止

- `take_snapshot` を使用しない（大量DOM出力による停止リスク）。

## 2. 代替手段

- 視覚確認: `take_screenshot`
- データ取得: `evaluate_script`（サンプリング/統計）
- 大量データ（10,000件以上）は段階的に取得する。

## 3. 運用

- 取得対象と件数上限を先に決めてから実行する。
- 重い操作は小さな単位に分割し、結果を確認しながら進める。
