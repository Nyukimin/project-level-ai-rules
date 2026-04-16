# CLAUDE フォルダ構造ガイド

## 1. 目的

このドキュメントは、`CLAUDE` 配下に置いている Claude 向けルール/Skill 構成の意味と運用方針を整理するためのガイドです。

## 2. ディレクトリの意味（このリポジトリ内の見方）

- `CLAUDE/user_home` は **`~/`（ユーザーホーム）相当のモック** です。ここにあるものは「マシン全体で共通の Claude 設定」として **`~` 直下に配置**します。
- `CLAUDE/user_home/ Project` は **個別リポジトリ（プロジェクトルート）相当のモック** です。フォルダ名の先頭にスペースがあるのはリポジトリ内の区別用であり、**実際にコピーするときは自分のプロジェクトのルートディレクトリ**（例: `~/src/my-app`）に置き換えます。

この 2 層を分けることで、グローバル方針とプロジェクト固有設定を分離して管理します。

## 3. Tree（ファイルの意味付き）

リポジトリ上のパスと、各ファイル・ディレクトリの役割です。

```text
CLAUDE/
├── README.md                          # 本ガイド（構成説明・コピー先の対応表）
└── user_home/                         # [モック] 実機では ~/ に相当
    ├── CLAUDE.md                      # ホーム直下の「最上位判断軸」（憲法の要約）
    └── .claude/
        └── rules/                     # 全プロジェクト共通の詳細ルール（番号は読み順・優先度の目安）
            ├── 00-global.md           # グローバル憲法の詳細（推論原則・禁止事項など）
            ├── 10-safety-priority.md  # 安全・優先順位
            ├── 20-investigation.md    # 調査・検証の進め方
            ├── 30-coding-and-architecture.md
            │                          # コーディング・アーキテクチャ
            └── 35-chrome-devtools-safety.md
                                       # Chrome DevTools 利用時の安全注意
    └──  Project/                      # [モック] 実機では各プロジェクトのルートに相当
        ├── .clauderules               # プロジェクト直下の Claude 向け短い方針（エントリ）
        └── .claude/
            ├── settings.local.json    # ローカル専用設定（共有・コミット対象から外す想定）
            ├── rules/                 # このプロジェクト専用ルール
            │   ├── 10-safety-priority.md
            │   ├── 20-investigation.md
            │   ├── 30-coding-and-architecture.md
            │   ├── 40-project.md      # プロジェクト固有情報・方針
            │   ├── 50-domain.md     # ドメイン知識・用語
            │   └── 60-physical-log.md # 物理ログ・観測の扱い
            └── skills/                # 手順を Skill 化したもの（必要に応じて起動）
                ├── analyze-codebase/
                │   ├── SKILL.md       # Skill の入口（目的・使い方）
                │   ├── phases/        # フェーズ別の手順
                │   └── templates/     # 出力テンプレート
                └── debug-investigate/
                    ├── SKILL.md
                    ├── checklists/
                    └── templates/
```

### 3.1 ルール番号について

- **`user_home/.claude/rules/`**: マシン全体で効かせたい共通ルール。
- **`user_home/ Project/.claude/rules/`**: 特定リポジトリだけに効かせたいルール（`40-project` / `50-domain` などはこちら側に置く想定）。

ファイル名の先頭番号は、読む順序やレイヤ分けの目安です（プロジェクト内ルールはグローバルと同名番号があっても、**プロジェクト側がそのリポジトリ文脈で上書き・追加**するイメージで運用します）。

## 4. ユーザーがどこにコピーすればよいか（明示）

**原則**: 上記 Tree の「モック」パスを、次の **実機パス** にコピー（またはマージ）します。

| このリポジトリ上のパス | 実機でのコピー先 | 内容の要約 |
|------------------------|------------------|------------|
| `CLAUDE/user_home/CLAUDE.md` | **`~/CLAUDE.md`** | ホーム直下の最上位方針 |
| `CLAUDE/user_home/.claude/` 以下すべて | **`~/.claude/`** 以下に対応配置 | グローバル共通ルール（`rules/` など） |
| `CLAUDE/user_home/ Project/.clauderules` | **`<あなたのプロジェクトルート>/.clauderules`** | プロジェクト用エントリ |
| `CLAUDE/user_home/ Project/.claude/` 以下すべて | **`<あなたのプロジェクトルート>/.claude/`** 以下に対応配置 | プロジェクト専用ルール・Skill・ローカル設定 |

### 4.1 コマンド例（Unix系）

ホーム向け（上書き注意。既存がある場合はバックアップ推奨）:

```bash
cp "/path/to/project-level-ai-rules/CLAUDE/user_home/CLAUDE.md" "$HOME/CLAUDE.md"
cp -R "/path/to/project-level-ai-rules/CLAUDE/user_home/.claude" "$HOME/"
```

プロジェクト向け（`<PROJECT_ROOT>` を実際のパスに置き換え）:

```bash
PROJECT_ROOT="/path/to/your/repo"
cp "/path/to/project-level-ai-rules/CLAUDE/user_home/ Project/.clauderules" "$PROJECT_ROOT/.clauderules"
cp -R "/path/to/project-level-ai-rules/CLAUDE/user_home/ Project/.claude" "$PROJECT_ROOT/"
```

**注意**:

- ` Project` フォルダ名の先頭スペースは **このリポジトリ内のモック用** です。コピー先では **通常のプロジェクトディレクトリ名** のみを使います。
- `settings.local.json` は **個人環境・秘密情報を含み得る** ため、チームで共有する場合はコミット対象から外すか、サンプルのみを置く運用にしてください。

## 5. ルール（Rules）の役割（補足）

### 5.1 `user_home/CLAUDE.md`

- Claude が毎回参照すべき最上位の判断軸（憲法レイヤー）を置きます。
- 例: 安全性優先、最小差分、禁止事項、作業フロー。

### 5.2 `user_home/.claude/rules/00-global.md` など

- `CLAUDE.md` の詳細版として、推論原則や運用規約を分離します。
- 共通ルールはここに集約し、`CLAUDE.md` の肥大化を防ぎます。

### 5.3 `user_home/ Project/.claude/rules/40-project.md` など

- リポジトリ固有の名前・制約・ドメインを書く場所です。テンプレートとして別名で複製して使っても構いません。

## 6. Skill の役割

### 6.1 `user_home/ Project/.claude/skills/`

- 反復的な手順（調査、解析など）を Skill として保持する領域です。

### 6.2 収録済み Skill（現状）

- `analyze-codebase`: コードベースの段階的解析
- `debug-investigate`: 仮説駆動の不具合調査

## 7. 運用方針

- 毎回必要な判断軸は `CLAUDE.md` に残す
- 詳細ルールは `.claude/rules/` に分離する
- 手順書は `.claude/skills/` に分離する
- ローカル固有の設定は `settings.local.json` に閉じる

## 8. 今後の整理方針（推奨）

- 新規 Skill は `SKILL.md` を最小単位として追加する
- 役割が重複するルールは `00-global.md` と各種ルールファイルへ再配置する

以上の方針で、Claude 用設定を「判断軸 / ルール / 手順 / ローカル差分」に分離して運用します。
