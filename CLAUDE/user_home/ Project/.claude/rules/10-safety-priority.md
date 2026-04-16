---
description: "組込みFW プロジェクトの安全ルール差分"
alwaysApply: true
---

# 10-safety-priority - プロジェクト差分

共通の安全ルールは `~/.claude/rules/10-safety-priority.md` を適用する。  
このファイルはプロジェクト固有の差分のみ定義する。

## 1. 既存コード保護

- `your-vendor-fw/` 配下の既存コードは原則変更しない。
- 変更が必要な場合は事前承認を取る。
- 動作確認済みモジュールの挙動変更は明示指示がある場合のみ許可。

## 2. ビルド・生成物

- 開発環境は macOS/Linux 前提、ビルドは `west build` を使用。
- `generated/` は直接編集しない（`.fbs` から再生成）。
- Kconfig は `.conf` で上書きし、Kconfig本体を直接改変しない。

## 3. 追加禁止事項

- West ワークスペース構造（`__west__/` など）の破壊。
- ステートマシン遷移ロジックの無断変更。
- `k_mem_slab` 解放漏れ（`k_mem_slab_free` 漏れ）。
