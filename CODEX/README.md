# CODEX フォルダ構造ガイド

## 1. 目的

`CODEX` 配下は、Codex 向けのルール配置をこのリポジトリ内で再現したものです。  
`CLAUDE` / `CURSOR` と**同じ内容・階層**を保ちつつ、Codex の主入口である **`AGENTS.md`** と **`skills/`** に合わせて再構成しています。

## 2. ディレクトリの意味（モックと実機の対応）

| このリポジトリ内のパス | 実機での意味 |
|------------------------|--------------|
| `CODEX/README.md` | 本ドキュメント（コピー不要） |
| `CODEX/user_home/` | **`~/`（ユーザーホーム）相当のモック**。ここにある `.codex/skills/` は **全プロジェクト共通** のルール Skill。 |
| `CODEX/user_home/ Project/` | **1 つのプロジェクトリポジトリのルート**相当。`AGENTS.md` と `skills/` は **そのプロジェクト専用**。 |

## 3. ツリー（ファイルの意味付き）

```text
CODEX/
├── README.md                          … 本ガイド
│
└── user_home/                         … 【モック】~/ （説明用）
    ├── .codex/
    │   └── skills/                    … 【グローバル】全プロジェクトで共有する共通 Skill
    │       ├── global-engineering-rules/
    │       │   └── SKILL.md
    │       └── modularization-rules/
    │           └── SKILL.md
    └──  Project/                      … 【モック】単一プロジェクトのルート
        ├── AGENTS.md                  … Codex が読むプロジェクトルートの主ルール（差分中心）
        └── skills/                    … このプロジェクト専用の Skill
            ├── analyze-codebase/
            │   ├── SKILL.md
            │   ├── phases/
            │   └── templates/
            └── debug-investigate/
                ├── SKILL.md
                ├── checklists/
                └── templates/
```

## 4. コピー先対応表

Codex では、**プロジェクトルートの `AGENTS.md` が主入口**です。  
ただし、このリポジトリでは `Claude` / `Cursor` の「ホーム配下の共通ルール」に対応するものとして、**`~/.codex/skills/` に共通 Skill を置く**運用にしています。

| コピー元（このリポジトリ） | コピー先（あなたの環境） | 説明 |
|----------------------------|--------------------------|------|
| `CODEX/user_home/.codex/skills/` 以下 | **`~/.codex/skills/`** | グローバル共通 Skill。複数プロジェクトで共有する判断軸。 |
| `CODEX/user_home/ Project/AGENTS.md` | **`<対象プロジェクトのルート>/AGENTS.md`** | Codex の最優先ルール。プロジェクト固有の前提・差分を置く。 |
| `CODEX/user_home/ Project/skills/` 以下 | **`<対象プロジェクトのルート>/skills/`** | そのリポジトリ専用の Skill。 |

### 4.1 コマンド例

```bash
REPO="/path/to/project-level-ai-rules"
PROJ="${HOME}/work/my-firmware"

mkdir -p "${HOME}/.codex/skills"
cp -R "${REPO}/CODEX/user_home/.codex/skills/." "${HOME}/.codex/skills/"
cp "${REPO}/CODEX/user_home/ Project/AGENTS.md" "${PROJ}/AGENTS.md"
mkdir -p "${PROJ}/skills"
cp -R "${REPO}/CODEX/user_home/ Project/skills/." "${PROJ}/skills/"
```

## 5. `~/.codex/skills/` と `skills/` の分担

- **`~/.codex/skills/`**:
  - `global-engineering-rules`: 00/10/20/30/35 相当の共通方針
  - `modularization-rules`: 40-modularization 相当の共通判断基準
- **`<repo>/skills/`**:
  - `analyze-codebase`, `debug-investigate` のように、プロジェクト用語・ディレクトリ構成・調査資産に依存する Skill
- **`<repo>/AGENTS.md`**:
  - プロジェクト固有の前提、ドメイン知識、ローカルな差分ルールを置く

## 6. CLAUDE / CURSOR との関係

- 方針・番号付け（`00` / `10` / `20` / `30` / `40` / `50` / `60` 相当）は `CLAUDE/` / `CURSOR/` と揃えています。
- Codex では `~/CLAUDE.md` のようなホーム直下ルールファイルではなく、**`~/.codex/skills/` に共通 Skill を置く**形でホーム側を表現しています。
- `AGENTS.md` は共通ルール全部を抱え込まず、**プロジェクト差分中心**にしています。
- Skill の内容は Claude 側の `.claude/skills/` と揃えていますが、配置を Codex 向けに `~/.codex/skills/` と `skills/` に振り分けています。
