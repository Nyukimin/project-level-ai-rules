# Codex Rules (Embedded FW Project)

この `AGENTS.md` は、Codex の**プロジェクト固有ルール**です。  
全プロジェクト共通の判断軸は `~/.codex/skills/` に置く前提とし、このファイルでは**このリポジトリ特有の前提・差分・ドメイン知識**を定義します。

## 先に使う共通 Skill

- `~/.codex/skills/global-engineering-rules/SKILL.md`
  - 安全性、調査、設計、実装、テスト、報告、Chrome DevTools 利用時の共通方針
- `~/.codex/skills/modularization-rules/SKILL.md`
  - モジュール分離や責務の切り出し判断

## プロジェクト前提

- `your-vendor-fw/` は実体コード（別リポジトリ）。
- `your-workspace/` は west ワークスペース（`your-vendor-fw/` を参照）。
- このリポジトリは主にドキュメント・設定・JTAG/Ozone管理を扱う。
- `your-app-src/` がファームウェア本体の主編集対象。
- `your-workspace/` / `your-app-src/` / `your-vendor-fw/` は利用者が差し替えるプレースホルダ。

## 技術スタック

- RTOS: Zephyr
- 言語: C
- ビルド: CMake + `west`
- ブートローダ: MCUboot
- デバッグ: SEGGER J-Link / Ozone

## プロジェクト安全差分

- `your-vendor-fw/` 配下の既存コードは原則変更しない。
- 変更が必要な場合は事前承認を取る。
- 動作確認済みモジュールの挙動変更は明示指示がある場合のみ許可。
- 開発環境は macOS/Linux 前提、ビルドは `west build` を使用。
- `generated/` は直接編集しない（`.fbs` から再生成）。
- Kconfig は `.conf` で上書きし、Kconfig本体を直接改変しない。
- West ワークスペース構造（`__west__/` など）の破壊を禁止する。
- ステートマシン遷移ロジックの無断変更を禁止する。
- `k_mem_slab` 解放漏れ（`k_mem_slab_free` 漏れ）を残さない。

## プロジェクト調査差分

- RTT は欠落・順不同前提で扱う。
- ANSIエスケープ除去と drop 区間明記を必須とする。
- Zephyr の uptime は絶対時刻ではない。RTC ログで補正する。
- `printk` 系ログは前後ログから時刻推定し、解析仕様を固定管理する。
- ログ不在を未実行と断定しない（early return / drop / filter を先に検証）。
- `err` でも正常扱いのイベント一覧を別管理する。
- 同一ログ多発時は、正当再実行とタイマー/ワーク漏れを時系列で切り分ける。
- C-L1: ログ無音時は「アプリ未起動」を先に疑う。
- C-L2: 計測点と観測対象レールの一致を確認する。
- C-L3: 異常判定は機械可読ルールで定義する。
- C-L4: ISR ログは非ブロッキング・短文に限定する。
- C-L5: ログ形式変更時はパーサ/可視化ツールを同時更新する。
- C-L6: 全モジュール DEBUG を禁止する。
- C-L7: 中央 dispatch は呼び出し元追跡可能にする。

## プロジェクト実装差分

- 命名は関数/変数 `snake_case`、定数 `UPPER_SNAKE_CASE`。
- モジュール単位で `src/{module}/` に分離する。
- エラーハンドリングはアサート依存にせず、リリース時動作も保証する。
- Zephyr ログ（`LOG_MODULE_REGISTER`, `LOG_ERR/WRN/INF/DBG`）を使用する。
- ステートマシン・永続化・BAT24/モータ設計は後述のドメインルールに従う。
- モジュール有効化は `CONFIG_APP_{MODULE}` + `add_subdirectory_ifdef` で制御する。
- テストは `test/` 配下に配置し、`west test` で実行する。
- `sm_dispatch` の呼び出しコンテキストを区別して確認する。
- `workq_h` / `sysworkq` のキュー詰まりを切り分ける。
- `immediate_stop` と後段 shutdown の責務分離を守る。
- タイマー停止漏れと再入条件を必ずレビューする。
- C-A1: 非同期入口に状態ガードを置く。
- C-A2: リソース生成/破棄の対応関係を管理する。
- C-A3: エラーパスでも解放漏れをなくす。
- C-A4: 設定値変更時の二次影響（CPU/電力/ログ量）を評価する。
- C-A5: レガシー定数の根拠を追跡可能にする。
- C-A6: HW操作は意図名ラッパーで抽象化する。
- C-A7: ワークキュー/ISR/タイマー相互作用をレビュー対象に含める。
- C-T1: テスト/デバッグコードをビルドフラグで隔離する。
- C-T2: 状態機械の「既に目標状態」をテストする。
- C-T3: 閾値・タイマ変更時は回帰テストを追加する。

## ドメイン固有ルール

### ステートマシン

- Dux ベースの一方向フローを維持する: `Action -> Reducer -> State -> Observer`
- Reducer は純粋関数とし、状態を直接破壊しない。
- `SM_REDUCER_REGISTER` / `SM_OBSERVER_REGISTER` / `SM_MIDDLEWARE_REGISTER` を使用する。
- アクションは `action_{module}_{name}` 形式で生成し、`sm_dispatch()` で流す。

### logger モジュール

- 種別は alert（`Persistent_ID_logger_alert`）と history（`Persistent_ID_logger_history`）。
- 追加は `logger_log_set()` + `action_logger_log_push()` を使う。
- alert は `action_logger_alert_push()` を使う。
- イベントカウンタは `action_logger_event_count()` / `action_logger_event_count_clear()` を使う。
- ログアップロードは `ready -> start -> send_try(loop) -> send_done -> up_done -> popped` の順序を守る。

### 永続化

- 永続化登録は `PERSISTENT_REGISTER` を使用する。
- `state_to_persistance` はシリアライズ、`persistance_to_action` は復元、`alternative_action` はエラー時代替を担当する。
- 復元時は root verify を行い、不正時は代替アクションにフォールバックする。
- 永続化制御状態（unknown/idle/revert/store）を明示する。

### FlatBuffers

- スキーマは `.fbs` で定義する。
- `flatcc` で `generated/` へ生成する。
- 生成物は直接編集しない。
- バイナリ契約（フィールド追加/互換性）を破壊しない。

### 依存関係

- 循環依存を作らない。
- `logger`, `motor`, `thumbturn` 等の中核モジュールは境界を明示して依存させる。
- モジュール間連携は action/event を優先し、暗黙共有を避ける。

### Ozone/JTAG/MCUboot

- `Erase All/EraseChip` は原則禁止（復旧時のみ明示承認）。
- 通常は `Erase Sectors` を使用する。
- アプリ書込みは署名済みhex、デバッグはelfを使い分ける。
- 署名鍵不一致時はブートローダ停止を疑う。

### RTT ログ運用

- RTT は欠落・並び替わり前提で扱う。
- ANSIコード除去、ドロップ区間明記を必須とする。
- Zephyr uptime は絶対時刻でない。RTCログで補正する。
- `printk` 系は時刻推定とフォーマット契約を別管理する。

### 起動/施解錠ログ解析

- 起動フローは「ブート -> 初期化 -> 状態遷移 -> サービス起動」の順で確認する。
- 施解錠は issuer・reason・電流値・モータ停止理由をセットで解釈する。
- `err` ログに正常イベントが混ざる運用があるため、一律異常判定しない。

### BAT24 制御

- BAT24 OFF までの経路を必ず追跡し、`MOTOR_OFF` 到達有無を確認する。
- current0 はモータ高側観測であり、BAT24残留の直接証拠にならない場合がある。
- BAT24残留の確認はGPIO等の直接観測を優先する。

### モータ停止シーケンス

- `StopReason` ごとの差異を区別して解析する。
- `current_task` 更新順序を守り、停止前後の整合性を維持する。
- `drv_motor_control_stop` 周辺の TOCTOU を常に疑う。
- `control_flag` の更新競合を避ける（原子性を意識）。

### sm_dispatch 用語・順序

- ディスパッチ層（呼び出し元 / キュー投入 / 実行）を分けて記録する。
- `k_poll_signal` を含む待機解除順序を固定し、再入条件を明示する。

### 電源急落調査フレーム

- シナリオID（S-A〜S-F）と持続機構分類（P/Q/R）で整理する。
- ユースケース（UC）を固定し、ケースごとに観測点を比較する。
- Driver経路番号を残して、どの経路で再現したか追跡可能にする。

### トレース設計

- 最小トレースポイントを固定し、毎回同じ観測粒度で採取する。
- タイムスタンプは uptime と wall-clock を併記する。
- 遅延イベント試験はベクタ化して再現可能にする。

### 追加ドメイン知識

- P-D1: `BAT_24V_ON` と `MOTOR_OFF` の同時Highは禁止。
- P-D2: モータ方向の真理値表を固定運用する。
- P-D3: autolock の fail_count リセット条件を明示管理する。
- P-D4: 施錠/解錠パラメータを分離して扱う。
- P-D5: `sleep_blocker` の設計根拠を維持する。
- P-D6: lifetest 開始前提条件を満たしてから実行する。
- P-D7: `locking -> unknown` 遷移の根本原因を分類管理する。
- P-D8: flatcc `emit_back` assert は再現条件と対処をセットで記録する。

## 物理ログ開発ルール

- 既存の動作確認済みコードは原則変更しない。
- 本フェーズは観測目的。バグを見つけても修正せず記録する。
- 新規実装より既存コード活用を優先する。
- 診断処理が製品動作へ影響しないことを最優先する。
- 新規実装は `src/physical_log/` に配置する。
- 既存コードへの変更は最小フック追加のみ許可。
- 動的メモリ確保は禁止。静的確保のみ。
- RAM使用量は必要最小限（目安: 70KB以下）。
- サンプリング間隔は 30秒固定、処理時間は軽量化（目安: 10ms以下）。
- 許可フックは `drv_overcurrent_sens.c -> physical_log_motor_event()`、`drv_gear_unlatch.c -> physical_log_gear_unlatch_event()`、初期化系 `-> physical_log_init()` のみ。
- 上記以外のフック追加は事前承認必須。
- バグ発見時は `docs/調査/` へ JST `YYYYMMDD_hhmmss_` プレフィックス必須で記録する。
- 本ルール内でバグ修正PRやワークアラウンド実装は行わない。
- 優先順位は 1.既存システム安定性 2.観測データ取得 3.実装のシンプルさ 4.効率化。
- P-P1: `[TL]` ログ契約（キー/単位/順序）をFWとツールで同期する。
- P-P2: G1（モータ区間残り値）とG2（真のIDLE）を混同しない。
- P-P3: BAT24残留は電流センサのみで断定せず、GPIO直接観測を優先する。
- P-P4: D1/D2/D3 はワークキュー経由で実行される前提で解析する。
- P-P5: 切り分け順序（A1/A2 -> E/F時系列）を固定する。
- P-P6: 累積カウンタは絶対値でなく差分で分析する。

## プロジェクト Skill の使い分け

- コードベース全体を段階的に整理・地図化したい場合は `skills/analyze-codebase/SKILL.md` を使う。
- バグ症状から仮説駆動で調査したい場合は `skills/debug-investigate/SKILL.md` を使う。
- これらの Skill はこのプロジェクトの用語・ログ規約・調査資産を前提にしているため、一般用途ではなくこのリポジトリ文脈で使う。
