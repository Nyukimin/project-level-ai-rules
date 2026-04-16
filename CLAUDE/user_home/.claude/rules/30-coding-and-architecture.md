---
description: "コーディング・設計・テストルール（共通）"
globs: "**/*.c,**/*.h,**/*.py,**/*.sh,**/*.js,**/*.ts,**/*.go,**/*.cc,**/*.cpp,**/*.html,**/*.css"
---

# 30-coding-and-architecture - 共通実装ルール

## 1. 設計プロセス（必須）

1. 要件を明確化（5W1H）。
2. 既存実装・既存APIを確認。
3. 選択肢とトレードオフを比較。
4. 受け入れ基準を先に定義。
5. 重要判断は ADR で記録。

## 2. 設計原則

- SOLID / DRY / KISS / YAGNI を優先する。
- 1モジュール1責務を基本とする。
- 拡張は追加で行い、既存シグネチャの破壊を避ける。
- エラーハンドリングは初期段階で設計する。

## 3. 並行性レビュー（必須）

- TOCTOU を避ける（判定と実行を分離しない）。
- ISR は短時間・非ブロッキング、重処理はワーカー側に寄せる。
- 停止APIは冪等にする。
- 二段階停止（immediate stop / 後段 shutdown）を分離する。
- タイマー start/stop の対を明示し、停止漏れを防ぐ。

## 4. エラーハンドリング

- 例外・エラーを握りつぶさない。
- リトライは有限回、指数バックオフ、対象限定で実装する。
- 初期化失敗は呼び出し元へ伝播し、失敗状態を明示する。

## 5. コーディング規約（要約）

- Python: `snake_case` / 型ヒント必須 / `ruff`。
- JS/TS: `camelCase` / `PascalCase` / `eslint` + `prettier` + `tsc --noEmit`。
- Go: `gofmt` + `go vet`（必要なら `golangci-lint`）。
- C/C++: `snake_case` + `UPPER_SNAKE_CASE` / `clang-format` / `clang-tidy`。
- 公開APIはコメント・Docstringを付与する。

## 6. Lint・テスト（必須）

- Lintエラーゼロを維持する。
- コミット前に Lint / Format / 型チェック / 関連テストを実施する。
- TDD（Red-Green-Refactor）、FIRST、AAA を推奨する。

## 7. 参照

- 共通方針: `00-global.md`
- 安全/禁止事項: `10-safety-priority.md`
- 調査/ログ: `20-investigation.md`
