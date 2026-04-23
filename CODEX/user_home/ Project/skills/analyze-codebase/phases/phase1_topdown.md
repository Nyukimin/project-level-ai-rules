# Phase 1: トップダウン解析 詳細手順

## 対象ソースコード

`$0`（デフォルト: `your-vendor-fw/src/`）

---

## Step 1-6: モジュール解析（並列実行可能）

### 実行方法

サブエージェントを起動し、各グループを解析する。
最大6つのサブエージェントを並列で起動可能。

各サブエージェントに渡すプロンプトの共通部分:

```
serena MCP で対象プロジェクトをアクティベートし、以下のモジュールを解析せよ。

## 解析手順
0. 【--refs 指定時のみ】関連する設計書を読み込み、以下を把握:
   - .md ファイル: Read ツールで直接読み込む
   - .pdf ファイル: 以下の2段階で読み込む
     a. Bash で `pdftotext <file> -` を実行しテキスト全文を取得
     b. 図が重要なページ（Phase 0 で特定済み、または「図」「レイヤー」「フロー」等のキーワードで推定）を
        Bash で `pdftoppm -png -r 200 -f N -l N <file> /tmp/pdf_<name>` で PNG 変換
     c. Read ツールで PNG を読み込み、図の内容（アーキテクチャ図、フロー図、状態遷移図等）を把握
   - 把握すべき情報:
     - 設計書が記述する「意図された仕様」
     - 図に含まれるアーキテクチャ・データフロー・状態遷移
     - 機能の要件・制約条件
   - この情報は「設計意図」「落とし穴」セクションに反映する
   - 仕様と実装の乖離（図との乖離を含む）が見つかれば「落とし穴・注意点」に記録する
1. `get_symbols_overview` でファイル内の関数一覧を取得
2. `find_symbol` で大関数（50行超）の body を取得
3. 大関数内の switch/case, if-else を分類・グループ化
4. `find_referencing_symbols` でモジュール間の関係を確認
5. `search_for_pattern` で暗黙のパターンを検出
   （fallthrough, 副作用のある分岐, エラーパス等）
6. 「地図型テンプレート」に従ってドキュメントを生成
   （役割・関係性・構造マップ・落とし穴を重視、シグネチャ転記は不要）

## テンプレート
（templates/module_analysis.md の内容に従う）

## 出力
Write ツールで docs/codebase-map/modules/ に書き出す。

## 外部資料（Phase 0 で自動マッピングされた設計書）
{refs_list}
※ 上記ファイルを Read で読み込み、設計意図・仕様との乖離の検出に活用すること
※ 外部資料がない場合はこのセクションは空
```

### Step 1: コア基盤モジュール

**対象**: `state_machine/`, `event/`, `action/`, `command/`

重点ポイント:
- Dux フレームワークの構造（SM_REDUCER_REGISTER, SM_OBSERVER_REGISTER, SM_MIDDLEWARE_REGISTER）
- sm_dispatch() の内部フロー
- SM_State の構造
- module_init() との関係

### Step 2: 通信認証モジュール

**対象**: `ble/`, `nfc_auth/`, `subsys/`

重点ポイント:
- BLE サービス/キャラクタリスティクスの構成
- NFC 認証フロー
- コマンド受信からアクション生成までのフロー

### Step 3: 物理制御モジュール

**対象**: `motor/`, `drivers/`, `hal/`, `thumbturn/`

重点ポイント:
- モーター制御シーケンス（施錠/解錠）
- ドライバ層の構成（overcurrent, gear_unlatch 等）
- サムターン検出の仕組み
- HAL の抽象化レベル

### Step 4: データ永続化モジュール

**対象**: `data/`, `logger/`, `physical_log/`, `shared/`

重点ポイント:
- PERSISTENT_REGISTER マクロと永続化フロー
- ログの種類（alert, history）と格納構造
- FlatBuffers スキーマとシリアライゼーション
- shared の共有ユーティリティ

### Step 5: 付帯機能モジュール

**対象**: `autolock/`, `button/`, `audio/`, `power/`

重点ポイント:
- オートロックのタイマー制御
- ボタン操作のイベント処理
- 音声フィードバックのパターン
- バッテリー管理と電源状態

### Step 6: テスト基盤モジュール

**対象**: `lifetest/`, `factory/`, `main.c`, `fatal.c`

重点ポイント:
- ライフテストのシナリオ構成
- 工場テストモードの制御
- main.c のブートシーケンス（module_init 呼び出し順）
- fatal エラーハンドリング

---

## Step 7: モジュール結合ポイントマップ

### 前提
Step 1-6 の出力が `docs/codebase-map/modules/` に存在すること。

### 手順
1. Step 1-6 の全出力を Read ツールで読み込む
2. 各モジュールの「外部依存」「被依存」テーブルをクロスリファレンス
3. Middleware/Reducer/Observer チェーンを優先度順に整理
4. SM_State のメンバーとモジュールの対応を整理
5. templates/integration_map.md に従ってドキュメント生成

### 出力
`docs/codebase-map/結合ポイントマップ.md`

---

## Step 8: ユースケース逆引きリファレンス

### 前提
Step 1-7 の出力が存在すること。

### 手順
1. Step 1-7 の出力を読み込む
2. 9つのユースケースそれぞれについて:
   a. トリガーを特定
   b. コードを追跡して処理フローを構築
   c. 各ステップの関数を `find_symbol` で確認
   d. templates/usecase_reference.md に従って記述
3. 不明箇所は serena のシンボル検索で補完

### 出力
`docs/codebase-map/ユースケース逆引き.md`

---

## Step 9: アーキテクチャ総合レポート

### 前提
Step 1-8 の出力が存在すること。

### 手順
1. 全出力を読み込み、以下を統合:
   - システムアーキテクチャ概要（1ページ）
   - 完全なモジュール依存図
   - 主要データフロー
   - 修正影響度チェックリスト
2. serena メモリにサマリを保存

### 出力
`docs/codebase-map/アーキテクチャ総合.md`
serena メモリ: `architecture_analysis_summary`
