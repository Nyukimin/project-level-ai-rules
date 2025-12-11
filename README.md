# project-level-ai-rules

# AI開発ルールセット

このリポジトリには、AI開発支援（Claude Code、Cursor等）で使用する共通ルールファイルが含まれています。

## 構成

```text
.
├── README.md                    # リポジトリの概要（このファイル）
├── .clauderules                 # Claude Code 用ルール参照（例）
├── .cursorrules                 # Cursor 用ルール参照（例）
└── rules/                       # ルール本体
    ├── common/
    │   ├── GLOBAL_AGENT.md
    │   ├── rules_architecture.md
    │   ├── rules_backend.md
    │   ├── rules_frontend.md
    │   ├── rules_logging.md
    │   ├── rules_security.md
    │   └── rules_testing.md
    ├── PROJECT_AGENT_sample.md   # プロジェクト固有ルール（テンプレ）
    └── rules_domain_sample.md    # ドメイン固有ルール（テンプレ）
```

## 使用方法

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

## 主要なルール内容

### GLOBAL_AGENT.md

- AI利用の共通方針（思考過程の明確な提示、作業の進め方）
- ペアプログラミングにおけるコア原則
- コード修正における思考憲法
- 推論・計画の8原則
- コーディング全般に関する共通ルール
- テスト・TDDに関する共通ルール
- 仕様決定プロセス
- Git運用
- セキュリティ
- モジュール安定性ルール
- 進捗・達成度の評価
- 調査記録の管理

### rules_backend.md

- Python、Go、C、C++のコーディングスタイル
- エラーハンドリング
- Lintingルール（必須）
- データ処理アプローチの選択原則

### rules_frontend.md

- HTML、CSS、JavaScript、TypeScriptのコーディングスタイル
- Linting・フォーマット（必須）

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

- **2025-12-11**: 初版公開
  - 共通ルールの集約と整理
  - プロジェクト固有ルールとの分離
  - 固有名詞の削除（Publicリポジトリ対応）
