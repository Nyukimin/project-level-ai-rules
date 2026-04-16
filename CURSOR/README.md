# CURSOR フォルダ構造ガイド

## 1. 目的

`CURSOR` 配下は、Cursor 用のルール配置（`~/.cursor/rules/` とプロジェクト `.cursor/rules/`）を**ディレクトリ構造ごと模したもの**です。  
`CLAUDE` 配下の Claude ルールと**同一の内容・階層**になるよう同期しています。

実運用では、プロジェクトルールは **`.cursor/rules/*.mdc`**（YAML フロントマター付き）が前提です。

## 2. ディレクトリの意味（モックと実機の対応）

| このリポジトリ内のパス | 実機での意味 |
|------------------------|--------------|
| `CURSOR/README.md` | 本ドキュメント（コピー不要） |
| `CURSOR/user_home/` | **`~/`（ユーザーホーム）相当**。ここにある `.cursor/rules/` は **全プロジェクト共通**のルール。 |
| `CURSOR/user_home/ Project/` | **1 つのプロジェクトリポジトリのルート**相当（名前にスペースあり。実際のプロジェクト名に読み替える）。中身は **そのプロジェクト専用**の差分ルール。 |

## 3. ツリー（ファイルの意味付き）

```text
CURSOR/
├── README.md                          … 本ガイド（リポジトリ内ドキュメント）
│
└── user_home/                         … 【モック】~/ （ホーム直下とみなす）
    ├── .cursor/
    │   └── rules/                     … 【グローバル】全プロジェクトで共有する憲法・共通ルール
    │       ├── 00-global.mdc          … 開発の共通方針（最優先）
    │       ├── 10-safety-priority.mdc … 安全・セキュリティ
    │       ├── 20-investigation.mdc … 調査・ログ・記録
    │       ├── 30-coding-and-architecture.mdc … 実装・設計・テスト
    │       └── 35-chrome-devtools-safety.mdc … Chrome DevTools MCP の安全運用
    │
    └──  Project/                      … 【モック】単一プロジェクトのルート（例: 組込みFWリポジトリ）
        ├── .cursorrules               … そのプロジェクトで @ 参照するルールの一覧（エントリ）
        └── .cursor/
            └── rules/                 … 【プロジェクト差分】グローバルに上乗せする内容
                ├── 10-safety-priority.mdc   … 安全ルールのプロジェクト差分
                ├── 20-investigation.mdc     … 調査・ログの差分
                ├── 30-coding-and-architecture.mdc … 実装ルールの差分
                ├── 40-project.mdc           … プロジェクト固有の前提・スタック
                ├── 50-domain.mdc            … ドメイン知識
                └── 60-physical-log.mdc      … 物理ログ等のドメイン別ルール
```

### 3.1 `.mdc` について

- 先頭の **YAML フロントマター**で `description` / `globs` / `alwaysApply` などを宣言します。
- 本文の相互参照は **`.mdc` ファイル名**で統一しています（調査メモなど別ファイルは `.md` のままの例もあります）。

## 4. コピー先対応表（どれをどこへ置くか）

次のとおり **左から右へコピー**します（上書きする前に既存のバックアップを推奨）。

| コピー元（このリポジトリ） | コピー先（あなたのマシン） | 説明 |
|----------------------------|----------------------------|------|
| `CURSOR/user_home/.cursor/rules/*.mdc` | **`~/.cursor/rules/`** | グローバルルール。Cursor がユーザースコープで読み込む想定の置き場所。 |
| `CURSOR/user_home/ Project/.cursor/rules/*.mdc` | **`<対象プロジェクトのルート>/.cursor/rules/`** | そのリポジトリだけに効かせる差分（番号 `40` 以降や差分上書き）。 |
| `CURSOR/user_home/ Project/.cursorrules` | **`<対象プロジェクトのルート>/.cursorrules`** | 任意。プロジェクトで `@` 参照したい場合のサンプル。下記「注意」を必ず読む。 |

### 4.1 コマンド例

**グローバル（ホーム）へ**

```bash
REPO="/path/to/project-level-ai-rules"   # このリポジトリの実パスに置き換え
mkdir -p ~/.cursor/rules
cp "${REPO}/CURSOR/user_home/.cursor/rules/"*.mdc ~/.cursor/rules/
```

**あるプロジェクトリポジトリへ（例: `~/work/my-firmware`）**

```bash
PROJ="${HOME}/work/my-firmware"          # あなたのプロジェクトルートに置き換え
mkdir -p "${PROJ}/.cursor/rules"
cp "${REPO}/CURSOR/user_home/ Project/.cursor/rules/"*.mdc "${PROJ}/.cursor/rules/"
# .cursorrules を使う場合のみ
cp "${REPO}/CURSOR/user_home/ Project/.cursorrules" "${PROJ}/.cursorrules"
```

※ パスにスペースが含まれる ` Project` は、シェルではクォートするか、ファインダーからコピーしてもよいです。

### 4.2 `.cursorrules` サンプルについての注意

`Project/.cursorrules` 内の **`@../.cursor/rules/*.mdc`** は、モックのディレクトリ配置専用です。

- モックでは `Project` の親が `user_home` で、その親に **グローバル**の `.cursor/rules/` があるため、`../.cursor/rules/` が「ホーム相当の共通ルール」を指せます。
- 実際に **プロジェクトだけを別パスに単独クローン**している場合、`../` は「親フォルダ」の `.cursor` を指すため、**意図した `~/.cursor/rules/` にならない**ことがあります。

**実運用の取り方（いずれか）**

1. **推奨**: グローバルは **`~/.cursor/rules/` にコピー済み**とし、Cursor のルール設定でユーザールールとして効くようにする。プロジェクトの `.cursorrules` では **`@.cursor/rules/*.mdc`（プロジェクト配下）だけ**を列挙するか、グローバル行を削る。
2. グローバルも `.cursorrules` で明示したい場合は、**自分のマシンのレイアウトに合わせて** `@` パスを書き換える（必要なら絶対パスや `~` の解釈が通る形にする）。

グローバル要約用の `~/CURSOR.md` は置かず、**`00-global.mdc` に統合**する方針です。

## 5. 運用の整理

- **グローバル**: `~/.cursor/rules/*.mdc` に共通ルール。
- **プロジェクト**: 各リポジトリの `.cursor/rules/*.mdc` に差分・ドメイン固有。
- **エントリ**: `.cursorrules` は任意。使う場合は `@` パスが実ディレクトリと一致するように調整する。

## 6. CLAUDE との関係

- 方針・番号付け（`00` / `10` / `20` …）は `CLAUDE/user_home` と揃えています。
- Claude 側は `.claude/rules/*.md`、Cursor 側は `.cursor/rules/*.mdc` と拡張子が異なりますが、**対応関係は番号とファイル名（拡張子除く）で一致**させています。
