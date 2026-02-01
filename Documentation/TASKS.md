# TASKS.md - プロジェクト進捗管理

## SESSION CHECKPOINT

**最終更新**: 2026-02-01
**フェーズ**: **実装開始準備完了**

### 最新セッション（2026-02-01）
- 機能一覧仕様書作成（PRD_04_Features.md v1.0）
- モックアップ・仕様書・TASKS.md整合性チェック完了（矛盾なし）
- 状態遷移仕様書作成（PRD_05_State_Transitions.md v1.1）
- Mermaid図形式への変換、spec-writerスキル更新
- Git初期化・GitHub連携（https://github.com/ryyo1125/AI-Knowledge-Hub）
- README.md作成・プッシュ
- CLAUDE.md拡張（3層実行ルール、完了条件チェックリスト追加）

---

## ✅ 決定済み事項

| ID | 課題 | 決定 |
|----|------|------|
| D-1 | Cron Job の実装方式 | **手動実行**（Phase 1） |
| D-2 | スクレイピングの実装方式 | **Cheerio + 対象サイト限定** |
| D-3 | マニュアル相互リンクの方式 | **手動選択**（Phase 1） |

---

## 📋 MVP実装タスク

**詳細**: [MVP_Implementation_Tasks.md](MVP_Implementation_Tasks.md) (v1.1)

| Phase | 内容 | タスク数 | 状態 |
|-------|------|----------|------|
| Phase 0 | 環境構築 | 5 | ⏳ 未着手 |
| Phase 1 | 基盤実装 | 9 | ⏳ 未着手 |
| Phase 2 | 収集機能 | 6 | ⏳ 未着手 |
| Phase 3 | 分析機能 | 6 | ⏳ 未着手 |
| Phase 4 | UI実装 | 10 | ⏳ 未着手 |
| Phase 5 | 統合・テスト | 4 | ⏳ 未着手 |
| **合計** | | **40** | |

---

## 📝 仕様書追加作成（実装前に推奨）

| ID | 仕様書 | 優先度 | 状態 |
|----|--------|--------|------|
| S-01 | PRD_05_State_Transitions.md（状態遷移仕様） | HIGH | ✅ 完了 |
| S-02 | PRD_06_Error_Handling.md（エラーハンドリング仕様） | HIGH | ⏳ 未着手 |
| S-03 | PRD_07_AI_Prompts.md（AI分析プロンプト仕様） | HIGH | ⏳ 未着手 |

> S-02, S-03 は実装開始後でも対応可能だが、Phase 3（AI分析機能）開始前には完了推奨

---

## ⏳ 次のタスク（Phase 0）

- [ ] **T-001**: Next.js プロジェクト初期化
- [ ] **T-002**: 依存パッケージのインストール
- [ ] **T-003**: Supabase プロジェクト作成
- [ ] **T-004**: 環境変数の設定
- [ ] **T-005**: ディレクトリ構成の作成

---

## ⛔ Blocking Tasks（人間の宿題）

- [ ] Supabaseアカウント作成（未作成の場合）
- [ ] Anthropic APIキー取得（未取得の場合）

---

## ✅ 完了タスク

- [x] 要件ブレインストーミング
- [x] 技術スタック決定（Next.js 14 + Supabase + Claude API）
- [x] 評価基準の定義（重要度/有用度/難易度 1-5）
- [x] プロジェクトルール策定
- [x] 要件定義書作成（PRD_01_Overview.md v1.1）
- [x] 技術仕様書作成（PRD_02_Technical.md v1.1）
- [x] セルフレビュー実施・仕様書改善
- [x] 決定待ち課題の解決（D-1〜D-3）
- [x] MVP実装タスク一覧作成（v1.0）
- [x] 外部AIフィードバック取り込み（v1.1）
- [x] SuperClaudeフレームワーク導入
- [x] カスタムAgent hook設定
- [x] 機能一覧仕様書作成（PRD_04_Features.md v1.0）
- [x] 仕様書・モックアップ整合性チェック
- [x] 状態遷移仕様書作成（PRD_05_State_Transitions.md v1.1）
- [x] spec-writerスキル更新（Mermaid必須化）

---

## 📚 ドキュメント一覧

| ファイル | 内容 | バージョン |
|----------|------|-----------|
| [PRD_01_Overview.md](Specifications/PRD_01_Overview.md) | 製品要件定義書（全体像） | v1.1 |
| [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) | 技術仕様書（DB/API/画面） | v1.1 |
| [PRD_03_UI_Specification.md](Specifications/PRD_03_UI_Specification.md) | UI仕様書（画面設計詳細） | v1.0 |
| [PRD_04_Features.md](Specifications/PRD_04_Features.md) | 機能一覧仕様書 | v1.0 |
| [PRD_05_State_Transitions.md](Specifications/PRD_05_State_Transitions.md) | 状態遷移仕様書 | v1.1 |
| [evaluation-criteria.md](evaluation-criteria.md) | 評価基準定義 | v1.0 |
| [MVP_Implementation_Tasks.md](MVP_Implementation_Tasks.md) | MVP実装タスク一覧 | v1.1 |
| [SuperClaude_Guide.md](SuperClaude_Guide.md) | SuperClaude活用ガイド | v1.0 |
| [mockup/](mockup/) | HTMLモックアップ | v1.0 |

---

## 🚀 次回セッションの開始点

1. `activate` コマンドでセッション開始
2. Blocking Tasks（Supabase/Anthropic）の確認
3. **Phase 0: T-001** から実装開始

### 推奨ツール
- `sc:implement` で機能実装
- `backend-architect` Agent でAPI/DB設計
- `frontend-developer` Agent でUI実装
