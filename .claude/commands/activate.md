---
name: activate
description: "セッション開始・プロジェクトアクティベーション"
shortcuts:
  - "activate"
tags:
  - bootstrap
  - activation
---

# OUTPUT LANGUAGE: 日本語 (Japanese)

**参照**: 詳細ルールは `CLAUDE.md` を参照

---

## Phase-A (Foundational Sync)

1. **Read-Constitution**: `CLAUDE.md` を読み込み、開発ルール再ロード
2. **Read-Checkpoint**: `Documentation/TASKS.md` の `SESSION CHECKPOINT` を特定
3. **Gate-Check**: Blocking Tasks が未完了なら停止
4. **Load-Prepare**: 関連 `TRUTH`・`skill` をリストアップ（読み込みは実行時）
5. **Report**: 現状、人間の宿題、AIの初動案を簡潔に提示

## Phase-B (Implementation)

- Phase-Aで特定した差分に基づき実装開始
- 仕様書優先の原則に基づきリソース取得

## Constraints

- **無目的な探索禁止**: TRUTH参照に基づく特定ファイル読み込みは許可
- Checkpointの依存関係を絶対遵守

## Output Format

```markdown
## セッション開始レポート

### 現在のチェックポイント
- 最終更新: [日時]
- 完了タスク: [リスト]
- 進行中タスク: [リスト]

### Blocking Tasks（人間の宿題）
- [ ] [タスク内容]

### AI初動案
1. [最初に行うべきアクション]
2. [次のアクション]

### 関連リソース
- 仕様書: [パス]
- 参照ファイル: [パス]
```
