# モジュール結合ポイントマップ テンプレート

以下のテンプレートに従って結合ポイントを分析すること。

---

## 1. アーキテクチャ概要

### Dux（Redux風）アーキテクチャ データフロー

（ASCII図でデータフローを記述）

```
Action → Middleware Chain → Reducer Chain → State → Observer Chain
```

### 状態ツリー構造

（SM_State のメンバーをツリー形式で記述）

## 2. モジュール間依存関係マトリクス

### 凡例
- **R**: Reducer 登録（SM_REDUCER_REGISTER）
- **O**: Observer 登録（SM_OBSERVER_REGISTER）
- **M**: Middleware 登録（SM_MIDDLEWARE_REGISTER）
- **A**: Action dispatch（sm_dispatch 経由）
- **D**: API 直接呼び出し

### コア基盤 → 他モジュール

| 呼び出し元 | 呼び出し先 | 種別 | なぜこの依存が必要か |
|-----------|-----------|------|-------------------|
| module_a | module_b | R/O/M/A/D | {この依存が存在する理由} |

（カテゴリごとに繰り返し）

## 3. Middleware チェーン（実行順）

| 優先度 | モジュール | 処理内容 |
|--------|-----------|---------|
| {priority} | {module} | {description} |

## 4. Reducer チェーン（実行順）

| 優先度 | モジュール | 処理対象 |
|--------|-----------|---------|
| {priority} | {module} | {description} |

## 5. Observer チェーン

| モジュール | 監視対象状態 | トリガー条件 |
|-----------|------------|------------|
| {module} | {state_member} | {condition} |
