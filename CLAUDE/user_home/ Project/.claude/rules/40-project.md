---
description: "組込みFW プロジェクト固有の開発ルール"
alwaysApply: true
---

# 40-project.md - 組込みFW プロジェクト固有ルール

**最終更新**: 2026-04-16  
**バージョン**: 2.2

## 1. プロジェクト前提

- `your-vendor-fw/` は実体コード（別リポジトリ）。
- `your-workspace/` は west ワークスペース（`your-vendor-fw/` を参照）。
- このリポジトリは主にドキュメント・設定・JTAG/Ozone管理を扱う。
- `your-app-src/` がファームウェア本体の主編集対象。
- `your-workspace/` / `your-app-src/` / `your-vendor-fw/` は利用者が差し替えるプレースホルダ。

## 2. 技術スタック

- RTOS: Zephyr
- 言語: C
- ビルド: CMake + `west`
- ブートローダ: MCUboot
- デバッグ: SEGGER J-Link / Ozone

## 3. コーディングルール（要点）

- 命名: 関数/変数 `snake_case`、定数 `UPPER_SNAKE_CASE`。
- モジュール単位で `src/{module}/` に分離する。
- エラーハンドリングはアサート依存にせず、リリース時動作も保証する。
- Zephyr ログ（`LOG_MODULE_REGISTER`, `LOG_ERR/WRN/INF/DBG`）を使用する。

## 4. 実装ルール

- ステートマシン・永続化・BAT24/モータ設計は `50-domain.md` に従う。
- モジュール有効化は `CONFIG_APP_{MODULE}` + `add_subdirectory_ifdef` で制御する。
- テストは `test/` 配下に配置し、`west test` で実行する。

## 5. 共通ルール参照

- 共通方針: `00-global.md`
- 安全/禁止事項: `10-safety-priority.md`
- 調査/ログ: `20-investigation.md`
- コーディング/設計/テスト: `30-coding-and-architecture.md`
- ドメイン固有: `50-domain.md`
- 物理ログ: `60-physical-log.md`

## 6. 追加ルール（P-A）

- P-A1: BAT24 は `_immidiate_stop` で切らず、後段 `MOTOR_OFF` で停止する設計を前提に分析する。
- P-A2: Ozone の `Erase All/EraseChip` は原則禁止。通常は `Erase Sectors` を使用する。
- P-A3: フラッシュは `firmware.signed.hex`、デバッグは `firmware.elf`。署名鍵不一致は起動失敗要因。
- P-A4: RTT開始時は PC位置、RTTシグネチャ、ブートローダ停滞、鍵一致を確認する。
- P-A5: `sm_dispatch` 実行コンテキストと `workq_h/sysworkq` の詰まりを切り分ける。
- P-A6: `your-app-root` と `your-vendor-fw/` は自動同期されない。意図的同期手順を必ず踏む。
- P-A7: ELFのDWARFパス不一致は path substitute かシンボリックリンクで補正する。

## 7. Chrome DevTools MCP

- 共通ルール `~/.claude/rules/35-chrome-devtools-safety.md` を適用する。
