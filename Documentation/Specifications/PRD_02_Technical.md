# AI Coding Knowledge Hub - 技術仕様書

**バージョン**: 1.1
**最終更新**: 2026-01-31
**関連**: [PRD_01_Overview.md](PRD_01_Overview.md)

---

## 1. システムアーキテクチャ

### 1.1 全体構成図

```
┌─────────────────────────────────────────────────────────────────┐
│                         Vercel                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Next.js 14 App                        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │   │
│  │  │   Pages     │  │    API      │  │  Cron Jobs      │  │   │
│  │  │  (React)    │  │   Routes    │  │  ※要決定        │  │   │
│  │  └─────────────┘  └──────┬──────┘  └────────┬────────┘  │   │
│  └───────────────────────────┼──────────────────┼───────────┘   │
└──────────────────────────────┼──────────────────┼───────────────┘
                               │                  │
          ┌────────────────────┼──────────────────┼────────────┐
          │                    ▼                  ▼            │
          │  ┌─────────────────────────────────────────────┐  │
          │  │              Supabase                        │  │
          │  │  ┌───────────┐  ┌───────────┐  ┌─────────┐  │  │
          │  │  │ PostgreSQL│  │   Auth    │  │ Storage │  │  │
          │  │  │    DB     │  │ (Phase2)  │  │(Phase2) │  │  │
          │  │  └───────────┘  └───────────┘  └─────────┘  │  │
          │  └─────────────────────────────────────────────┘  │
          │                                                    │
          │  ┌─────────────────────────────────────────────┐  │
          │  │           External Services                  │  │
          │  │  ┌───────────┐  ┌───────────┐               │  │
          │  │  │ Claude    │  │   RSS     │               │  │
          │  │  │  API      │  │  Feeds    │               │  │
          │  │  └───────────┘  └───────────┘               │  │
          │  └─────────────────────────────────────────────┘  │
          └────────────────────────────────────────────────────┘
```

### 1.2 データフロー

```
[RSS Feeds / Web Pages]
        │
        ▼ (Cron: 1回/日)
┌───────────────────┐
│  Collector Job    │ ← 記事取得・パース
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Deduplication    │ ← URL正規化 + タイトル類似度チェック
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│    AI Analyzer    │ ← Claude API 呼び出し（リトライあり）
└─────────┬─────────┘
          │ (失敗時: status='analysis_failed')
          ▼
┌───────────────────┐
│  Database Save    │ ← articles テーブルに保存
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Human Review     │ ← Web UI で確認・承認
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Report Generator  │ ← 週次レポート生成
└───────────────────┘
```

---

## 2. データベース設計

### 2.1 ER図

```
┌─────────────────┐       ┌─────────────────┐
│     sources     │       │    articles     │
├─────────────────┤       ├─────────────────┤
│ id (PK)         │───┐   │ id (PK)         │
│ name            │   └──<│ source_id (FK)  │
│ url             │       │ url             │
│ feed_url        │       │ title           │
│ type            │       │ content         │
│ is_active       │       │ summary         │
│ scrape_config   │       │ published_at    │
│ created_at      │       │ collected_at    │
│ updated_at      │       │ ai_importance   │
└─────────────────┘       │ ai_usefulness   │
                          │ ai_difficulty   │
                          │ ai_reason       │
                          │ ai_tags         │
                          │ human_importance│
                          │ human_usefulness│
                          │ human_difficulty│
                          │ human_comment   │
                          │ status          │
                          │ created_at      │
                          │ updated_at      │
                          └────────┬────────┘
                                   │
┌─────────────────┐                │
│    manuals      │                │
├─────────────────┤                │
│ id (PK)         │                │
│ title           │                │
│ content         │                │
│ tags            │                │
│ difficulty      │                │
│ related_manual_ids ←─────────────┤ (manuals同士の相互参照)
│ source_article_id ───────────────┘
│ status          │
│ created_at      │       ┌─────────────────┐
│ updated_at      │       │     reports     │
└─────────────────┘       ├─────────────────┤
                          │ id (PK)         │
                          │ title           │
                          │ content         │
                          │ period_start    │
                          │ period_end      │
                          │ article_ids[]   │←─ (articles参照)
                          │ status          │
                          │ created_at      │
                          │ updated_at      │
                          └─────────────────┘
```

### 2.2 テーブル定義

#### sources（監視対象サイト）

```sql
CREATE TABLE sources (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  url TEXT NOT NULL UNIQUE,
  feed_url TEXT,  -- RSSフィードURL（あれば）
  type TEXT NOT NULL DEFAULT 'rss',  -- 'rss' | 'scrape'
  is_active BOOLEAN NOT NULL DEFAULT true,

  -- スクレイピング設定（type='scrape'の場合）
  scrape_config JSONB,
  -- {
  --   "titleSelector": "h1.article-title",
  --   "contentSelector": "div.article-body",
  --   "dateSelector": "time.published",
  --   "dateFormat": "YYYY-MM-DD"
  -- }

  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

#### articles（収集記事）

```sql
CREATE TABLE articles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source_id UUID NOT NULL REFERENCES sources(id) ON DELETE CASCADE,
  url TEXT NOT NULL UNIQUE,
  title TEXT NOT NULL,

  -- 本文: AI分析用に最大5000文字まで保存
  content TEXT CHECK (char_length(content) <= 5000),
  summary TEXT,  -- AI生成要約（100文字以内）

  published_at TIMESTAMPTZ,
  collected_at TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- AI評価
  ai_importance INTEGER CHECK (ai_importance BETWEEN 1 AND 5),
  ai_usefulness INTEGER CHECK (ai_usefulness BETWEEN 1 AND 5),
  ai_difficulty INTEGER CHECK (ai_difficulty BETWEEN 1 AND 5),
  ai_reason JSONB,  -- {importance: "...", usefulness: "...", difficulty: "..."}
  ai_tags TEXT[] CHECK (array_length(ai_tags, 1) <= 5),
  ai_analyzed_at TIMESTAMPTZ,  -- 分析完了日時
  ai_error TEXT,  -- エラー時のメッセージ

  -- 人間評価
  human_importance INTEGER CHECK (human_importance BETWEEN 1 AND 5),
  human_usefulness INTEGER CHECK (human_usefulness BETWEEN 1 AND 5),
  human_difficulty INTEGER CHECK (human_difficulty BETWEEN 1 AND 5),
  human_comment TEXT,
  reviewed_at TIMESTAMPTZ,  -- レビュー日時

  -- ステータス
  -- pending: 未レビュー
  -- approved: 承認済み
  -- rejected: 却下
  -- analysis_failed: AI分析失敗（リトライ対象）
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'approved', 'rejected', 'analysis_failed')),

  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- インデックス
CREATE INDEX idx_articles_status ON articles(status);
CREATE INDEX idx_articles_collected_at ON articles(collected_at DESC);
CREATE INDEX idx_articles_source_id ON articles(source_id);
CREATE INDEX idx_articles_ai_importance ON articles(ai_importance DESC);
```

#### manuals（マニュアル）

```sql
CREATE TABLE manuals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT NOT NULL,  -- Markdown形式
  tags TEXT[],
  difficulty INTEGER CHECK (difficulty BETWEEN 1 AND 5),

  -- 関連マニュアル（手動選択）
  related_manual_ids UUID[],

  -- 元記事（オプション）
  source_article_id UUID REFERENCES articles(id) ON DELETE SET NULL,

  status TEXT NOT NULL DEFAULT 'draft'
    CHECK (status IN ('draft', 'published')),

  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_manuals_status ON manuals(status);
CREATE INDEX idx_manuals_tags ON manuals USING GIN(tags);
```

#### reports（週次レポート）

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT NOT NULL,  -- Markdown形式
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  article_ids UUID[],  -- 含まれる記事ID
  status TEXT NOT NULL DEFAULT 'draft'
    CHECK (status IN ('draft', 'published')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_reports_period ON reports(period_start DESC);
```

---

## 3. API設計

### 3.1 エンドポイント一覧

| Method | Endpoint | 説明 | 戻り値ポリシー |
|--------|----------|------|---------------|
| GET | `/api/sources` | 監視対象一覧 | DTO（※100件以下想定） |
| POST | `/api/sources` | 監視対象追加 | ACTION |
| PUT | `/api/sources/[id]` | 監視対象更新 | ACTION |
| DELETE | `/api/sources/[id]` | 監視対象削除 | ACTION |
| GET | `/api/articles` | 記事一覧 | PAGINATED |
| GET | `/api/articles/[id]` | 記事詳細 | DTO |
| PUT | `/api/articles/[id]/review` | 人間レビュー | ACTION |
| POST | `/api/articles/[id]/reanalyze` | AI再分析 | ACTION |
| POST | `/api/collect` | 手動収集実行 | ACTION |
| GET | `/api/reports` | レポート一覧 | PAGINATED |
| POST | `/api/reports/generate` | レポート生成 | ACTION |
| GET | `/api/manuals` | マニュアル一覧 | PAGINATED |
| POST | `/api/manuals` | マニュアル作成 | ACTION |

### 3.2 戻り値ポリシー定義

#### ACTION（状態変更系）

```typescript
interface ActionResponse<T = unknown> {
  success: boolean;
  message: string;
  data?: T;
  error?: {
    code: string;
    details?: string;
  };
}

// 成功例
{
  success: true,
  message: "記事のレビューを保存しました",
  data: { id: "xxx-xxx" }
}

// 失敗例
{
  success: false,
  message: "AI分析に失敗しました",
  error: {
    code: "AI_ANALYSIS_ERROR",
    details: "Rate limit exceeded"
  }
}
```

#### DTO（データ取得系）

```typescript
// 例: Article
interface ArticleDTO {
  id: string;
  sourceId: string;
  sourceName: string;
  url: string;
  title: string;
  summary: string | null;
  publishedAt: string | null;
  collectedAt: string;
  aiScore: {
    importance: number;
    usefulness: number;
    difficulty: number;
    reason: {
      importance: string;
      usefulness: string;
      difficulty: string;
    };
    tags: string[];
    analyzedAt: string;
  } | null;
  humanScore: {
    importance: number | null;
    usefulness: number | null;
    difficulty: number | null;
    comment: string | null;
    reviewedAt: string | null;
  };
  status: 'pending' | 'approved' | 'rejected' | 'analysis_failed';
}
```

#### PAGINATED（一覧取得系）

```typescript
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

// 例
{
  data: [...],
  pagination: {
    total: 150,
    page: 1,
    limit: 20,
    totalPages: 8,
    hasNext: true,
    hasPrev: false
  }
}
```

---

## 4. 画面設計

### 4.1 画面一覧

| 画面 | パス | 機能 |
|------|------|------|
| ダッシュボード | `/` | 統計サマリー、最近の記事 |
| 記事一覧 | `/articles` | 記事の検索・フィルター・レビュー |
| 記事詳細 | `/articles/[id]` | 記事内容・レビュー入力 |
| ソース管理 | `/sources` | 監視対象の追加・編集・削除 |
| レポート一覧 | `/reports` | 週次レポートの閲覧・生成 |
| マニュアル一覧 | `/manuals` | マニュアルの閲覧・作成（Phase 2） |

### 4.2 画面ワイヤーフレーム

#### ダッシュボード

```
┌─────────────────────────────────────────────────────────────┐
│  AI Coding Knowledge Hub                        [Settings]  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │ 今週の収集  │ │ 未レビュー  │ │ 承認済み    │           │
│  │    45件     │ │    12件     │ │    33件     │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 最近の記事                               [収集実行]  │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ ★★★★☆ Claude Code新機能：...          [Review]     │   │
│  │ ★★★☆☆ AIコーディングのベスト...      [Review]     │   │
│  │ ★★★★★ 【重要】APIの破壊的変更...     [Approved]   │   │
│  │ ⚠️    分析失敗：タイムアウト...        [再分析]     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 記事レビュー画面

```
┌─────────────────────────────────────────────────────────────┐
│  ← 戻る                                                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Claude Code新機能：MCPサーバー連携                        │
│  Source: Zenn | Published: 2026-01-30                      │
│  [🔗 元記事を開く]                                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ AI要約                                               │   │
│  │ Claude CodeにMCPサーバー機能が追加され...           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ AI評価              │ 人間評価                      │   │
│  │ 重要度: ★★★★☆ (4) │ 重要度: [1][2][3][4][5]      │   │
│  │ 有用度: ★★★★☆ (4) │ 有用度: [1][2][3][4][5]      │   │
│  │ 難易度: ★★★☆☆ (3) │ 難易度: [1][2][3][4][5]      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  コメント:                                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  [却下] [承認]                                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. 外部連携

### 5.1 Claude API

#### リクエスト仕様

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface AnalysisResult {
  importance: number;
  importance_reason: string;
  usefulness: number;
  usefulness_reason: string;
  difficulty: number;
  difficulty_reason: string;
  summary: string;
  tags: string[];
}

// 分析リクエスト（リトライ付き）
const analyzeArticle = async (
  article: { title: string; content: string; url: string },
  retryCount = 0
): Promise<AnalysisResult> => {
  const MAX_RETRIES = 3;
  const RETRY_DELAYS = [1000, 5000, 30000]; // ms

  try {
    const response = await anthropic.messages.create({
      model: "claude-3-haiku-20240307",
      max_tokens: 1024,
      messages: [{
        role: "user",
        content: buildAnalysisPrompt(article)
      }]
    });

    const text = response.content[0].type === 'text'
      ? response.content[0].text
      : '';
    return JSON.parse(text);

  } catch (error) {
    if (retryCount < MAX_RETRIES && isRetryableError(error)) {
      await sleep(RETRY_DELAYS[retryCount]);
      return analyzeArticle(article, retryCount + 1);
    }
    throw error;
  }
};

const isRetryableError = (error: unknown): boolean => {
  // Rate limit, timeout, 5xx errors
  if (error instanceof Anthropic.RateLimitError) return true;
  if (error instanceof Anthropic.APIConnectionError) return true;
  if (error instanceof Anthropic.InternalServerError) return true;
  return false;
};
```

#### コスト見積もり

| 項目 | 値 |
|------|-----|
| 入力トークン単価 | $0.25 / 1M tokens |
| 出力トークン単価 | $1.25 / 1M tokens |
| 1記事あたり入力 | ~2,000 tokens |
| 1記事あたり出力 | ~500 tokens |
| 1記事あたりコスト | ~$0.001 |
| 週50記事 | ~$0.05 |
| 月200記事 | ~$0.20 |

**注意**: リトライにより最大3倍のコストが発生する可能性あり

---

## 6. Cron Job設計

> ⚠️ **要決定**: Vercel Hobby プランでは Cron 使用不可。代替手段を選択する必要あり。
> → [TASKS.md](../TASKS.md) の決定待ち課題を参照

### 6.1 スケジュール（案）

| Job | スケジュール | 説明 | タイムゾーン |
|-----|-------------|------|-------------|
| collect-articles | `0 21 * * *` (UTC) | 記事収集（JST 6:00） | UTC |
| generate-report | `0 0 * * 1` (UTC) | 週次レポート生成（JST 月曜9:00） | UTC |

### 6.2 実装オプション

```
オプションA: GitHub Actions（無料）
  - .github/workflows/collect.yml でスケジュール実行
  - Vercel の API Route を HTTP で呼び出し

オプションB: 外部サービス（無料枠あり）
  - cron-job.org / EasyCron 等
  - Webhook で API Route を呼び出し

オプションC: 手動実行のみ（Phase 1）
  - ダッシュボードに [収集実行] ボタン
  - 自動化は Phase 2 へ先送り
```

---

## 7. 重複検出ロジック

### 7.1 URL正規化

```typescript
const normalizeUrl = (url: string): string => {
  const parsed = new URL(url);

  // クエリパラメータを削除（トラッキング系）
  const trackingParams = ['utm_source', 'utm_medium', 'utm_campaign', 'ref'];
  trackingParams.forEach(p => parsed.searchParams.delete(p));

  // 末尾スラッシュを統一
  let path = parsed.pathname.replace(/\/+$/, '');

  return `${parsed.host}${path}${parsed.search}`.toLowerCase();
};
```

### 7.2 重複判定ロジック

```typescript
const isDuplicate = async (
  newArticle: { url: string; title: string; sourceId: string }
): Promise<boolean> => {
  const normalizedUrl = normalizeUrl(newArticle.url);

  // 1. URL完全一致チェック
  const urlMatch = await db.articles.findFirst({
    where: { url: normalizedUrl }
  });
  if (urlMatch) return true;

  // 2. 同一ソース内でのタイトル類似度チェック（閾値90%）
  const recentArticles = await db.articles.findMany({
    where: {
      sourceId: newArticle.sourceId,
      collectedAt: { gte: subDays(new Date(), 7) }
    },
    select: { title: true }
  });

  for (const article of recentArticles) {
    if (similarity(newArticle.title, article.title) > 0.9) {
      return true;
    }
  }

  return false;
};
```

---

## 8. セキュリティ考慮事項

### 8.1 API キー管理

```env
# .env.local (Gitにコミットしない)
ANTHROPIC_API_KEY=sk-ant-xxx
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx  # サーバーサイドのみ
```

### 8.2 Phase 2での認証

```typescript
// middleware.ts (Phase 2で追加)
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs';
import { NextRequest, NextResponse } from 'next/server';

export async function middleware(req: NextRequest) {
  const res = NextResponse.next();
  const supabase = createMiddlewareClient({ req, res });
  const { data: { session } } = await supabase.auth.getSession();

  if (!session) {
    return NextResponse.redirect(new URL('/login', req.url));
  }

  return res;
}

export const config = {
  matcher: ['/((?!login|api/cron|_next/static|favicon.ico).*)']
};
```

---

## 9. 成功指標の測定方法

### 9.1 AI分析精度の計算

```typescript
// 「一致」の定義: 各スコアの差が1以内
const isScoreMatch = (
  ai: { importance: number; usefulness: number; difficulty: number },
  human: { importance: number; usefulness: number; difficulty: number }
): boolean => {
  return (
    Math.abs(ai.importance - human.importance) <= 1 &&
    Math.abs(ai.usefulness - human.usefulness) <= 1 &&
    Math.abs(ai.difficulty - human.difficulty) <= 1
  );
};

// 精度計算（レビュー済み記事のうち一致した割合）
const calculateAccuracy = async (): Promise<number> => {
  const reviewed = await db.articles.findMany({
    where: {
      status: { in: ['approved', 'rejected'] },
      human_importance: { not: null }
    }
  });

  const matches = reviewed.filter(a => isScoreMatch(
    { importance: a.ai_importance, usefulness: a.ai_usefulness, difficulty: a.ai_difficulty },
    { importance: a.human_importance!, usefulness: a.human_usefulness!, difficulty: a.human_difficulty! }
  ));

  return matches.length / reviewed.length;
};
```

---

## 10. 承認履歴

| 日付 | バージョン | 承認者 | 内容 |
|------|-----------|--------|------|
| 2026-01-31 | 1.0 | - | 初版作成 |
| 2026-01-31 | 1.1 | - | セルフレビュー反映（エラーハンドリング、重複検出、タイムゾーン等） |
