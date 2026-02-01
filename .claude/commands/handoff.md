---
name: handoff
description: "セッション終了プロトコル"
shortcuts:
  - "handoff"
tags:
  - handoff
  - end-session
---

# OUTPUT LANGUAGE: 日本語 (Japanese)

---

## Goal

今回セッションの経過を記録し、次回セッションの初動をスムーズに繋げる。

---

## Handoff Protocol

### 1. TASKS.md 更新

- **SESSION CHECKPOINT**: 最上部に今回完了作業を記録
- **Active Tasks**: 人間/AIタスクを優先順位付けでリストアップ（核心を短文で）
- **TRUTH形式**:
  - `ファイルパス` (行番号) 例: `src/app/page.tsx` (L137-160)
  - `ファイルパス` (バージョン) 例: `Feature_02_Technical.md` (v2.0)

### 1.1 ファイルサイズ管理（重要）

**TASKS.mdは小さく保つ**:

- **残すべき内容**:
  - 最新SESSION CHECKPOINT（1-2個まで）
  - アクティブタスク（未完了タスク）
- **Archiveに移動すべき内容**:
  - 古いSESSION CHECKPOINT（3個以上前）
  - 完了済みタスク詳細
- **目安**: TASKS.mdは200行以下を目標

### 2. 完了報告

セッション終了時の出力形式:

```markdown
## セッション終了レポート

### 今回の成果
- [完了したタスク]
- [作成・更新したファイル]

### 次回の開始点
- Priority HIGH: [最優先タスク]
- Priority MEDIUM: [次点タスク]

### Blocking Tasks（人間の宿題）
- [ ] [必要なアクション]

### 技術的メモ
- [次回に引き継ぐべき技術的発見]
```

---

## 判断フロー

```
発見内容は？
├─ プロジェクト固有のロジック → 仕様書
├─ 一時的なバグ/TODO → TASKS.md
├─ 汎用的な技術知見 → knowledge-keeper で判断
└─ 不明 → 「推論：」付きでTASKS.mdに記録
```
