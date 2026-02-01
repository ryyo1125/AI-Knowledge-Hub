# SuperClaude活用ガイド

**プロジェクト**: AI Coding Knowledge Hub
**最終更新**: 2026-02-01

---

## 概要

SuperClaudeフレームワークを活用して、開発効率と品質を向上させる。

---

## フェーズ別推奨ツール

### Phase 0: 環境構築

```bash
# ワークフロー生成
/sc:workflow "Phase 0環境構築タスクを実行"

# 実装
/sc:implement "Next.jsプロジェクト初期化"
```

### Phase 1: 基盤実装

```bash
# 設計
/sc:design "Supabaseスキーマ設計" --think

# 実装（バックエンド）
# → backend-architect Agentを活用

# セキュリティレビュー
# → security-engineer Agentを活用
```

### Phase 2-4: 機能実装

```bash
# 機能実装
/sc:implement "RSS収集機能" --task-manage

# テスト
/sc:test

# 問題解決
/sc:troubleshoot "エラー内容"
```

### Phase 5: 統合・テスト

```bash
# 品質改善
/sc:improve --focus quality

# 分析
/sc:analyze --focus security
```

---

## 日常的な使用パターン

### セッション開始

```bash
/activate
# SuperClaudeが自動的にプロジェクトコンテキストを読み込み
```

### 実装作業

```bash
# 単純な実装
/sc:implement "機能名"

# 複雑な実装（タスク管理モード自動有効化）
/sc:implement "複雑な機能" --task-manage --think
```

### 問題発生時

```bash
# トラブルシューティング
/sc:troubleshoot "エラーメッセージ"

# 深い分析が必要な場合
/sc:troubleshoot "問題" --think-hard
```

### セッション終了

```bash
/handoff
# TASKS.mdが自動更新される
```

---

## Agent活用シナリオ

### Supabase設計時

```
Task agent: backend-architect
プロンプト: "articlesテーブルのRLS設計をレビューして"
```

### UI実装時

```
Task agent: frontend-architect
プロンプト: "記事一覧コンポーネントの設計を提案して"
```

### セキュリティ確認時

```
Task agent: security-engineer
プロンプト: "API保護ミドルウェアをレビューして"
```

### テスト戦略時

```
Task agent: quality-engineer
プロンプト: "RSS収集機能のテスト戦略を提案して"
```

---

## フラグ早見表

| 状況 | フラグ |
|------|--------|
| 要件が曖昧 | `--brainstorm` |
| 複雑な判断 | `--think` |
| アーキテクチャ | `--think-hard` |
| 長いセッション | `--uc` |
| 多ステップ作業 | `--task-manage` |
| 複数ツール連携 | `--orchestrate` |

---

## プロジェクト固有ルール

1. **日本語出力**: すべての出力は日本語
2. **仕様書参照**: 実装前に`Documentation/Specifications/`を確認
3. **進捗更新**: タスク完了時に`TASKS.md`を更新
4. **品質確認**: 実装後に`/check-project`を実行

---

## トークン効率化シンボル

長いセッションでは以下のシンボルを活用：

```
進捗: ✅ ❌ ⚠️ 🔄 ⏳
技術: ⚡性能 🛡️セキュリティ 🔧設定
論理: → ⇒ ∴ ∵
フロー: » 順次実行
```

例：
```
T-001 ✅ → T-002 🔄 → T-003 ⏳
API設計 ✅ » 実装 🔄 » テスト ⏳
```
