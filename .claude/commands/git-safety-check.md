# Git安全性確認コマンド

このコマンドはGitコミット前の安全性確認を行います。

## 実行手順

### 1. 機密情報チェック

以下のパターンでファイル内容を検索し、機密情報が含まれていないか確認してください：

```bash
# APIキー・シークレットのパターン検索
grep -r -i -E "(api[_-]?key|secret|password|token|credential)" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.json" --include="*.md" .

# .envファイルの存在確認（コミットしてはいけない）
ls -la .env* 2>/dev/null

# Supabase/Anthropic関連の直接記述確認
grep -r -E "(sk-ant-|sb-|supabase\.co)" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.json" .
```

### 2. ファイル一覧確認

```bash
# 全ファイル一覧（除外対象含む）
git status --porcelain

# 追加予定ファイル確認
git diff --cached --name-only
```

### 3. 大きなファイルチェック

```bash
# 1MB以上のファイル検出
find . -type f -size +1M -not -path "./.git/*"
```

### 4. 確認チェックリスト

コミット前に以下を確認：

- [ ] `.env`ファイルがコミット対象に含まれていない
- [ ] APIキー・シークレットがハードコードされていない
- [ ] `node_modules/`が除外されている
- [ ] 画像・動画など大きなバイナリファイルが含まれていない
- [ ] `.gitignore`が正しく設定されている

### 5. 安全なコミット実行

```bash
# 安全確認後のコミット手順
git add .
git status  # 最終確認
git commit -m "type: message"
```

---

## 緊急時の対応

### コミット後に機密情報を発見した場合

```bash
# 直前のコミットを取り消し（pushする前）
git reset --soft HEAD~1

# 特定ファイルをステージから除外
git reset HEAD <file>
```

### pushした後に発見した場合

1. 即座にAPIキーを無効化（Supabase/Anthropicダッシュボード）
2. 新しいキーを発行
3. git履歴の書き換えは複雑なため、キー無効化を優先
