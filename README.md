# project-level-ai-rules

## AI開発ルールセット

このリポジトリには、AI 開発支援（Claude Code、Cursor、Codex など）で使う**ルール・手順・判断軸**を、Git で管理しやすい形でまとめています。

## このプロジェクトの意味と意義

- **意味**: 「どの AI ツールにもその場しのぎで渡す」のではなく、**チームや個人で再利用できるルール資産**としてバージョン管理するための置き場です。ホーム直下の共通設定と、リポジトリ単位の差分を分けて考えられるようにしています。
- **意義**: ツールごとに**配置パス・ファイル形式（例: `CLAUDE.md` / `.mdc` / Skill）が異なる**ため、同一の方針でも「そのツール向けにコピーできる形」として並べ、**運用ドリフト**（Cursor だけ古い、など）を減らすことを目的とします。

## Claude 用と Cursor 用は別フォルダです

このリポジトリでは次のように分離しています。

| フォルダ | 対象 | 中身のイメージ |
|----------|------|----------------|
| [`CLAUDE/`](CLAUDE/README.md) | Claude Code / Claude 向け | `~/CLAUDE.md`、`~/.claude/rules/`、プロジェクトの `.clauderules` / `.claude/` など |
| [`CURSOR/`](CURSOR/README.md) | Cursor 向け | `~/.cursor/rules/*.mdc`、プロジェクトの `.cursorrules` / `.cursor/rules/` など |
| [`CODEX/`](CODEX/README.md) | Codex 向け | プロジェクトの `AGENTS.md`、`skills/`、必要に応じて `~/.codex/skills/` など |

**内容の階層や方針は揃える前提**ですが、**ファイル名・拡張子・置き場所はツール仕様どおりに別管理**します。混在させず、使うツールに対応するフォルダだけを参照・コピーしてください。

## 使うときに最初に読むもの（各フォルダの README）

操作手順・ツリー・コピー先はツールごとに異なるため、**必ず次の README を起点にしてください。ルートのこのファイルだけでは配置の詳細は書ききれません。**

- **Claude 系を使う場合**: [`CLAUDE/README.md`](CLAUDE/README.md)
- **Cursor を使う場合**: [`CURSOR/README.md`](CURSOR/README.md)
- **Codex を使う場合**: [`CODEX/README.md`](CODEX/README.md)
- **複数ツールを使う場合**: **各ツールの README** を読み、それぞれの実機パスへコピーする（同じ内容を1か所にマージするのではなく、**ツール別の既定ディレクトリ**に配置する想定です）。

## 構成（リポジトリの見方）

```text
.
├── README.md                 # 本ファイル（プロジェクト全体の位置づけ・入口）
├── CLAUDE/
│   └── README.md             # Claude 向け: ツリー・コピー先・運用（必読）
├── CURSOR/
│   └── README.md             # Cursor 向け: ツリー・コピー先・運用（必読）
├── CODEX/
│   └── README.md             # Codex 向け: ツリー・コピー先・運用（必読）
├── LICENSE
└── …                         # その他（ツール設定のスタブ等）
```

各 `README.md` 配下には、`user_home/`（ホーム相当）と ` Project/`（1 プロジェクトのルート相当）の**モック構造**が続きます。詳細は上記リンク先を参照してください。

## 使用方法（任意: プロジェクト直下に `rules/` を置く方式）

以下は、**プロジェクトルートに `rules/common/*.md` を置き、`.clauderules` または `.cursorrules` から `@` 参照する**従来パターンの説明です。Claude / Cursor の**推奨ディレクトリレイアウト**は [`CLAUDE/README.md`](CLAUDE/README.md) と [`CURSOR/README.md`](CURSOR/README.md) を優先してください。

### 1. プロジェクトへの導入

各プロジェクトのルートディレクトリに `rules/` ディレクトリをコピーし、`.clauderules` または `.cursorrules` から参照します。

```
your-project/
├── .clauderules          # または .cursorrules
└── rules/                # このリポジトリからコピー
    └── common/
        ├── GLOBAL_AGENT.md
        ├── rules_backend.md
        └── rules_frontend.md
```

### 2. ルールファイルの参照

`.clauderules`または`.cursorrules`に以下のように記述します：

```markdown
# 共通ルールの参照
@rules/common/GLOBAL_AGENT.md
@rules/common/rules_backend.md
@rules/common/rules_frontend.md

# プロジェクト固有ルール（このリポジトリでは sample を編集して運用）
@rules/PROJECT_AGENT_sample.md
@rules/rules_domain_sample.md
```

### 3. プロジェクト固有ルールの追加（このリポジトリの推奨）

このリポジトリでは、各プロジェクトごとのルールは **`rules/PROJECT_AGENT_sample.md` と `rules/rules_domain_sample.md` を編集して運用**します。
（プロジェクトごとにファイル名を変える場合は、参照側の `@rules/...` も合わせて更新してください）

```
your-project/
└── rules/
    ├── common/              # 共通ルール（このリポジトリから）
    ├── PROJECT_AGENT_sample.md
    └── rules_domain_sample.md
```

`.clauderules`または`.cursorrules`に追加：

```markdown
# 共通ルールの参照
@rules/common/GLOBAL_AGENT.md
@rules/common/rules_backend.md

# プロジェクト固有ルールの参照
@rules/PROJECT_AGENT_sample.md
@rules/rules_domain_sample.md
```

## ルールの階層構造

```
common/GLOBAL_AGENT.md（共通ルール - 厚い）
  ↓ 参照
各プロジェクト/PROJECT_AGENT.md（プロジェクト固有 - 薄い）
  ↓ 参照
各プロジェクト/rules_domain.md（ドメイン固有 - 薄い）
```

### 設計方針

- **共通ルール（厚い）**: 全プロジェクトで適用される一般的な原則を集約
- **プロジェクト固有ルール（薄い）**: プロジェクト固有の制約とドメイン知識のみを記載

## 主要なルール内容（`rules/common/` を使う場合）

プロジェクトに `rules/common/` を置いている場合の、**ファイル役割の一覧**です（[`CLAUDE/README.md`](CLAUDE/README.md) / [`CURSOR/README.md`](CURSOR/README.md) で述べるツール別配置とは別レイヤの「役割別分割」の説明です）。

### `rules/common/GLOBAL_AGENT.md`

- AIエージェント運用の**憲法**（全プロジェクト共通の前提・禁止事項）
- コミュニケーション方針（不確実点の扱い、根拠提示、説明責任）
- ペアプログラミング原則／コード修正の思考憲法／推論・計画の原則
- 変更管理（Git運用の最小ルール）、セキュリティ・ログ・調査記録の概要

### `rules/common/rules_architecture.md`

- 仕様検討〜設計〜技術選定の進め方（5W1H、受け入れ基準、トレードオフ）
- 設計原則（SOLID / KISS / YAGNI、再利用、DRY）
- リトライ・バッチ・設定管理・ADR（意思決定記録）の実践指針

### `rules/common/rules_backend.md`

- バックエンド実装の共通規約（Python/Go/C/C++）
- エラーハンドリング、例外設計、リトライ戦略、ログ出力指針
- Lint/フォーマットの必須運用（例: ruff、ESLint等への接続も含む）
- データ処理アプローチ選択（事前定義 vs 事後クレンジング）の判断基準

### `rules/common/rules_frontend.md`

- フロントエンド実装の共通規約（HTML/CSS/JavaScript/TypeScript）
- Lint/フォーマット/型チェックの必須運用（ESLint/Prettier/tsc 等）

### `rules/common/rules_logging.md`

- ログと調査記録の運用（保存場所、命名規則、Git管理との関係）
- 非破壊ログ、重複ログ防止、重要ログの扱い、機密情報マスキング

### `rules/common/rules_security.md`

- 機密情報の扱い（`.env`、認証情報、Gitに残さない原則）
- `.gitignore` テンプレ、依存関係の脆弱性チェック、権限設定、入力バリデーション

### `rules/common/rules_testing.md`

- テスト/品質保証の実践レシピ（TDD、FIRST/AAA、テスト構成・命名）
- カバレッジ目安、E2E方針、ブラウザ操作テスト時の注意、CIでの扱い

## 対象言語

- **Python** (.py)
- **JavaScript** (.js)
- **TypeScript** (.ts)
- **Go** (.go)
- **C** (.c)
- **C++** (.cc, .cpp)
- **Bash/Shell** (.sh)
- **YAML** (.yaml, .yml)
- **JSON** (.json)
- **HTML** (.html)
- **Dockerfile**
- **Makefile**

## ライセンス

このルールセットは、AI開発支援のためのガイドラインとして公開されています。自由に使用・改変してください。

## 更新履歴

- **2026-04-23**: `CODEX/` を追加し、`AGENTS.md` / `skills/` ベースの Codex 向け構成を追加
- **2026-04-16**: プロジェクトの位置づけ、`CLAUDE/` と `CURSOR/` の分離、各フォルダ README を入口とする旨を追記
- **2025-12-11**: 初版公開
  - 共通ルールの集約と整理
  - プロジェクト固有ルールとの分離
  - 固有名詞の削除（Publicリポジトリ対応）
