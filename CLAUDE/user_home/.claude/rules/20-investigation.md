---
description: "ログ・観測・調査記録ルール（共通）"
globs: "docs/**,logs/**,tools/**"
---

# 20-investigation - 共通調査ルール

## 1. 基本方針

- ログは原因再現、調査記録は再実行可能性を目的に残す。
- 機密情報を含めず、構造化・機械可読で保存する。
- ログ追加は非破壊（挙動変更なし、低遅延）で行う。

## 2. 調査記録（必須）

- 保存先: `docs/調査/`
- ファイル名: **JST `YYYYMMDD_hhmmss_` プレフィックス必須**
  - 例: `20260416_131500_起動失敗調査.md`
- 記載項目: 概要 / 発生条件 / 手順 / 事実 / 結論 / 関連ファイル

## 3. ログ保存

- 保存先: `logs/YYYYMMDD_hhmmss_説明/`
- `logs/` と `*.log` はGit管理外（`.gitignore`）にする。
- ログフォーマットにはバージョンを持たせる。

## 4. 重要ログと重複防止

- `audit` `state_change` `external` `business_logic` `error` `exception` `startup` `config` はサンプリング対象外。
- 同一イベントの多重記録を避ける（どの層で1回記録するかを固定）。
- デバッグログと監査ログを混在させない。

## 5. 機密マスキング

- `password` / `token` / `api_key` / `secret` / 個人識別子 / カード番号は必ずマスク。
- 構造化データでもキー名ベースで再帰マスクする。

## 6. ログレベル方針

- `DEBUG`: 開発時のみ
- `INFO`: 通常動作
- `WARNING`: 要注意だが継続可能
- `ERROR`: 処理失敗
- `CRITICAL`: 継続困難

本番は `INFO` 以上を基本とし、`DEBUG` は限定的に有効化する。

## 7. 参照

- 共通方針: `00-global.md`
- 安全/禁止事項: `10-safety-priority.md`
