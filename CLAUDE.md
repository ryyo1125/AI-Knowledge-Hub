# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference Index

**For AI Agent**:
- Session Start → `.claude/commands/activate.md`
- Session End → `.claude/commands/handoff.md`
- Spec Writing → Skill `spec-writer`
- Code Quality → `.claude/commands/check-project.md`

---

## PROJECT LANGUAGE SETTING

- **言語設定**: **日本語 (Japanese)**
- 原則としてすべての出力を日本語で行う。技術用語は英語可。

## CORE PRINCIPLES

1. **Spec-First**: 仕様書を元にコードを生成。正解は `Documentation/Specifications/` にある
2. **Task Truth**: `Documentation/TASKS.md` を唯一の進捗管理ソース。終了時に `handoff` で更新
3. **Fact-Based**: 推測排除。推論時は「推論：」と明記。技術課題はウェブ検索実行
4. **Concise Output**: 核心を端的に記載、人間の負荷最小化

## DEVELOPMENT WORKFLOW

### 三位一体の同期ワークフロー

1. **Checkpoint Sync**: `TASKS.md` の `SESSION CHECKPOINT` 読み込み、開始位置特定
2. **Spec Proposal**: コード修正前に仕様書修正案を提示・承認。`spec-writer` スキル使用
3. **Implementation**: 承認仕様に基づき実装。推論補完厳禁
4. **Review**: 修正コードと仕様書ロジックを対比報告し承認を得る

## TECH STACK

| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | Next.js 14 (App Router) | React Server Components |
| Backend | Next.js API Routes | Edge Runtime可 |
| Database | Supabase (PostgreSQL) | Row Level Security |
| AI | Claude API (Haiku) | 分析・要約用 |
| Hosting | Vercel | 自動デプロイ |
| Styling | Tailwind CSS | |

## TRUTH SOURCES

| 用途 | パス |
|------|------|
| 進捗管理 | `Documentation/TASKS.md` |
| 仕様書 | `Documentation/Specifications/` |
| 評価基準 | `Documentation/evaluation-criteria.md` |

## SKILLS

| スキル | 用途 |
|--------|------|
| `spec-writer` | 仕様書作成・更新時 |
| `knowhow-editor` | ルール/スキル作成・更新時 |
| `knowledge-keeper` | ナレッジ管理・永続化判断時 |

## SKILL WORKFLOW

```
仕様書変更 → spec-writer → 人間承認 → 実装 → コード生成
ルール変更 → knowhow-editor → 人間承認 → 更新実行
```

**並列起動禁止**: 1タスクに1スキル。連携時は順次実行。

## COMMANDS

人間が実行。AI自動実行禁止。

| コマンド | 用途 |
|----------|------|
| `activate` | セッション開始・プロジェクトアクティベーション |
| `handoff` | セッション終了 |
| `check-project` | コード品質チェック |

## PROJECT RULES

### 開発プロセス
- 進め方: 機能ごとに確認しながら進行
- 確認タイミング: 実装前後に確認
- 質問の仕方: 判断に迷ったら確認

### コード品質
- コメント: 日本語
- 変数・関数名: 英語（camelCase）
- エラーメッセージ: 日本語
- テスト: 重要部分のみ

### Git運用
- コミットメッセージ: 日本語
- コミット粒度: 機能単位
- ブランチ戦略: feature branch

### ドキュメント
- README: 詳細に
- 設計書: 作成する
- 変更履歴: Gitログで管理

### フォルダ管理
- フォルダ増加抑制: 新規フォルダ作成は最小限に
- 適切な配置: ファイルは内容に応じた既存フォルダに配置
- 配置先の判断基準:
  - ソースコード → `src/` 配下の適切なサブフォルダ
  - 仕様書・設計書 → `Documentation/Specifications/`
  - 進捗・タスク → `Documentation/`
  - 設定ファイル → プロジェクトルート

### AI指示
- 説明の詳しさ: 簡潔に
- 判断の委任: 大枠のみ確認
- 提案: 積極的に提案

---

## 実行ルール（3層構造）

AIの自律性レベルを明確化し、適切な確認ポイントを設定する。

### Layer 1: 人間確認必須 🔴

以下の操作は**必ず人間の承認を得てから**実行する。

| 操作 | 理由 |
|------|------|
| Supabaseスキーマ変更 | データ構造への影響大 |
| 環境変数の追加・変更 | セキュリティ・設定影響 |
| 外部API呼び出し設定 | コスト・セキュリティ |
| package.jsonの依存追加 | プロジェクト全体への影響 |
| 認証・認可ロジック変更 | セキュリティクリティカル |
| 本番環境への操作 | 取り消し困難 |

### Layer 2: 条件付き確認 🟡

以下の場合は**実行前に概要を報告し、承認を得る**。

| 条件 | 対応 |
|------|------|
| 3ファイル以上の同時変更 | 変更概要を提示 |
| 既存APIのレスポンス形式変更 | 影響範囲を説明 |
| 仕様書との乖離が発生 | 乖離内容と理由を報告 |
| 新規コンポーネント作成 | 配置場所と設計を確認 |

### Layer 3: 自律実行 🟢

以下は**報告のみで自律的に実行可能**。

- テスト追加・修正
- コンポーネント実装（仕様書通り）
- lint/format修正
- コメント・ドキュメント更新
- 既存パターンに従ったコード追加
- バグ修正（影響範囲が明確な場合）

---

## 完了条件チェックリスト

タスク完了時に以下を確認する。

### 機能実装時

- [ ] 仕様書の要件をすべて満たしている
- [ ] TypeScript型エラーなし（`npm run type-check`）
- [ ] Lintエラーなし（`npm run lint`）
- [ ] 関連テストがパス
- [ ] エラーハンドリング実装済み
- [ ] 日本語コメント追加済み

### API実装時

- [ ] 仕様書のエンドポイント定義と一致
- [ ] リクエスト/レスポンス型が定義済み
- [ ] エラーレスポンスが統一フォーマット
- [ ] Supabase RLSポリシー確認済み

### UI実装時

- [ ] モックアップと一致
- [ ] レスポンシブ対応（モバイル/デスクトップ）
- [ ] ローディング状態の実装
- [ ] エラー状態の実装
- [ ] アクセシビリティ基本対応

### セッション終了時

- [ ] TASKS.md更新済み
- [ ] 未完了タスクの記録
- [ ] 次回開始点の明記
- [ ] コミット済み（必要な場合）

## ENVIRONMENT

- **OS**: Windows 10/11
- **Runtime**: Node.js 20+
- **Package Manager**: npm

---

## SUPERCLOUD INTEGRATION

### フレームワーク有効化

SuperClaudeフレームワークをこのプロジェクトで活用する。

### 推奨フラグ

| フラグ | 用途 | 使用タイミング |
|--------|------|---------------|
| `--think` | 構造化分析 | 複雑な実装判断時 |
| `--think-hard` | 深い分析 | アーキテクチャ設計時 |
| `--brainstorm` | 要件探索 | 新機能検討時 |
| `--task-manage` | タスク管理 | 3ステップ以上の作業時 |
| `--uc` | トークン効率化 | 長いセッション時 |

### 推奨sc:コマンド

| コマンド | 用途 |
|----------|------|
| `sc:implement` | 機能実装 |
| `sc:test` | テスト実行 |
| `sc:troubleshoot` | 問題診断 |
| `sc:improve` | コード改善 |
| `sc:workflow` | ワークフロー生成 |
| `sc:design` | 設計作成 |

### 推奨Agent

| Agent | 用途 |
|-------|------|
| `backend-architect` | API/DB設計 |
| `frontend-architect` | UI/Component設計 |
| `security-engineer` | セキュリティレビュー |
| `quality-engineer` | 品質・テスト戦略 |

### モード活用

| モード | トリガー | 用途 |
|--------|----------|------|
| Task Management | 3+ステップ作業 | タスク階層管理 |
| Orchestration | 複数ツール連携 | 効率的なツール選択 |
| Introspection | エラー発生時 | 問題分析・学習 |

### シンボル活用（Token Efficiency）

```
状態: ✅完了 ❌失敗 ⚠️警告 🔄進行中 ⏳待機
技術: ⚡性能 🛡️セキュリティ 🔧設定 📦デプロイ
論理: → 導出 ⇒ 変換 ∴ 結論 ∵ 理由
```

### プロジェクト固有の統合ルール

1. **日本語優先**: SuperClaudeの出力も日本語で行う
2. **仕様書連携**: `sc:implement`時は`Documentation/Specifications/`参照必須
3. **進捗同期**: タスク完了時は`TASKS.md`も更新
4. **品質ゲート**: `sc:test`後に`check-project`実行推奨

---

## CUSTOM AGENTS

### 利用可能なAgent

| Agent | 専門領域 | 自動起動トリガー |
|-------|----------|-----------------|
| `backend-architect` | API設計、DB、スケーラビリティ | API, database, schema, バックエンド |
| `frontend-developer` | React、UI、アクセシビリティ | コンポーネント, UI, React, 画面 |
| `typescript-mcp-expert` | MCP Server開発 | MCP, transport, SDK |

### Agent活用ルール

**自動推奨**: hookにより関連キーワード検出時にAgent推奨を表示

**積極的活用**: 以下の場合はTaskツールでAgentを起動

```
# バックエンド設計時
Task agent: backend-architect
→ API設計、DBスキーマ、サービス境界定義

# フロントエンド実装時
Task agent: frontend-developer
→ Reactコンポーネント、状態管理、アクセシビリティ

# MCP開発時
Task agent: typescript-mcp-expert
→ MCPサーバー構築、ツール実装、トランスポート設定
```

### 本プロジェクトでの活用シナリオ

| タスク | 推奨Agent |
|--------|-----------|
| Supabase API Routes設計 | `backend-architect` |
| 記事一覧/詳細画面実装 | `frontend-developer` |
| レビュー画面UI | `frontend-developer` |
| Claude API統合設計 | `backend-architect` |
| 将来のMCP拡張 | `typescript-mcp-expert` |
