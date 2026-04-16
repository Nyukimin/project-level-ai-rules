---
description: "ドメイン固有知識（SM・BAT24・モータ・ログ解析等、組込みFW向け）"
globs: "your-workspace/**"
---

# 50-domain.md - ドメイン固有ルール

**最終更新**: 2026-04-16  
**バージョン**: 2.2

## 1. ステートマシン原則（必須）

- Dux ベースの一方向フローを維持する:
  - Action -> Reducer -> State -> Observer
- Reducer は純粋関数とし、状態を直接破壊しない。
- `SM_REDUCER_REGISTER` / `SM_OBSERVER_REGISTER` / `SM_MIDDLEWARE_REGISTER` を使用する。
- アクションは `action_{module}_{name}` 形式で生成し、`sm_dispatch()` で流す。

## 2. logger モジュール（必須）

- 種別:
  - alert (`Persistent_ID_logger_alert`)
  - history (`Persistent_ID_logger_history`)
- 追加:
  - `logger_log_set()` + `action_logger_log_push()`
  - alert は `action_logger_alert_push()`
- イベントカウンタ:
  - `action_logger_event_count()` / `action_logger_event_count_clear()`
- ログアップロードは以下順序を守る:
  - ready -> start -> send_try(loop) -> send_done -> up_done -> popped

## 3. 永続化（必須）

- 永続化登録は `PERSISTENT_REGISTER` を使用する。
- API責務:
  - `state_to_persistance`: シリアライズ
  - `persistance_to_action`: 復元
  - `alternative_action`: エラー時代替
- 復元時は root verify を行い、不正時は代替アクションにフォールバックする。
- 永続化制御状態（unknown/idle/revert/store）を明示する。

## 4. FlatBuffers（必須）

- スキーマは `.fbs` で定義する。
- `flatcc` で `generated/` へ生成する。
- 生成物は直接編集しない。
- バイナリ契約（フィールド追加/互換性）を破壊しない。

## 5. 依存関係ルール

- 循環依存を作らない。
- `logger`, `motor`, `thumbturn` 等の中核モジュールは境界を明示して依存させる。
- モジュール間連携は action/event を優先し、暗黙共有を避ける。

## 6. Ozone/JTAG/MCUboot

- `Erase All/EraseChip` は原則禁止（復旧時のみ明示承認）。
- 通常は `Erase Sectors` を使用する。
- アプリ書込みは署名済みhex、デバッグはelfを使い分ける。
- 署名鍵不一致時はブートローダ停止を疑う。

## 7. RTT ログ運用

- RTT は欠落・並び替わり前提で扱う。
- ANSIコード除去、ドロップ区間明記を必須とする。
- Zephyr uptime は絶対時刻でない。RTCログで補正する。
- `printk` 系は時刻推定とフォーマット契約を別管理する。

## 8. 起動/施解錠ログ解析の要点

- 起動フローは「ブート -> 初期化 -> 状態遷移 -> サービス起動」の順で確認する。
- 施解錠は issuer・reason・電流値・モータ停止理由をセットで解釈する。
- `err` ログに正常イベントが混ざる運用があるため、一律異常判定しない。

## 9. BAT24 制御ルール（必須理解）

- BAT24 OFF までの経路を必ず追跡し、`MOTOR_OFF` 到達有無を確認する。
- current0 はモータ高側観測であり、BAT24残留の直接証拠にならない場合がある。
- BAT24残留の確認はGPIO等の直接観測を優先する。

## 10. モータ停止シーケンス（必須）

- `StopReason` ごとの差異を区別して解析する。
- `current_task` 更新順序を守り、停止前後の整合性を維持する。
- `drv_motor_control_stop` 周辺の TOCTOU を常に疑う。
- `control_flag` の更新競合を避ける（原子性を意識）。

## 11. sm_dispatch 用語・順序

- ディスパッチ層（呼び出し元 / キュー投入 / 実行）を分けて記録する。
- `k_poll_signal` を含む待機解除順序を固定し、再入条件を明示する。

## 12. 電源急落調査フレーム

- シナリオID（S-A〜S-F）と持続機構分類（P/Q/R）で整理する。
- ユースケース（UC）を固定し、ケースごとに観測点を比較する。
- Driver経路番号を残して、どの経路で再現したか追跡可能にする。

## 13. トレース設計

- 最小トレースポイントを固定し、毎回同じ観測粒度で採取する。
- タイムスタンプは uptime と wall-clock を併記する。
- 遅延イベント試験はベクタ化して再現可能にする。

## 14. 追加ドメイン知識（P-D）

- P-D1: `BAT_24V_ON` と `MOTOR_OFF` の同時Highは禁止。
- P-D2: モータ方向の真理値表を固定運用する。
- P-D3: autolock の fail_count リセット条件を明示管理する。
- P-D4: 施錠/解錠パラメータを分離して扱う。
- P-D5: `sleep_blocker` の設計根拠を維持する。
- P-D6: lifetest 開始前提条件を満たしてから実行する。
- P-D7: `locking -> unknown` 遷移の根本原因を分類管理する。
- P-D8: flatcc `emit_back` assert は再現条件と対処をセットで記録する。

## 15. 参照

- プロジェクト固有ルール: `40-project.md`
- 物理ログ調査ルール: `60-physical-log.md`
