# MVP実装タスク一覧

**バージョン**: 1.1
**作成日**: 2026-01-31
**更新日**: 2026-01-31（外部レビュー反映）
**目標**: MVP（Phase 1）の完成

---

## 📊 全体サマリー

| フェーズ | タスク数 | 見込み |
|----------|----------|--------|
| Phase 0: 環境構築 | 5 | - |
| Phase 1: 基盤実装 | **9** | - |
| Phase 2: 収集機能 | **7** | - |
| Phase 3: 分析機能 | **5** | - |
| Phase 4: UI実装 | 10 | - |
| Phase 5: 統合・テスト | 4 | - |
| **合計** | **40** | - |

> **v1.1 変更点**: 外部レビューを反映し、API保護・コンテンツ取得強化・コスト制御タスクを追加

---

## 📚 参照仕様書一覧

| 仕様書 | パス | 参照タスク |
|--------|------|-----------|
| 製品要件定義書 | [PRD_01_Overview.md](Specifications/PRD_01_Overview.md) | 全タスク（プロジェクト全体像） |
| 技術仕様書 | [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) | T-005〜T-033（DB/API/画面設計） |
| UI仕様書 | [PRD_03_UI_Specification.md](Specifications/PRD_03_UI_Specification.md) | T-010〜T-033（UI/UX詳細設計） |
| 評価基準定義 | [evaluation-criteria.md](evaluation-criteria.md) | T-021, T-022（AI分析プロンプト） |
| HTMLモックアップ | [mockup/](mockup/) | T-010〜T-033（UI実装の参考） |

---

## Phase 0: 環境構築

### [T-001] Next.js プロジェクト初期化

**ゴール**: 開発環境の基盤を構築

```bash
npx create-next-app@latest ai-knowledge-hub --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

**設定項目**:
- [x] TypeScript
- [x] Tailwind CSS
- [x] ESLint
- [x] App Router
- [x] src/ ディレクトリ
- [x] import alias (@/*)

**成果物**: 動作する Next.js プロジェクト
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 1（技術スタック）

---

### [T-002] 依存パッケージのインストール

**ゴール**: 必要なライブラリを追加

```bash
# Supabase
npm install @supabase/supabase-js @supabase/ssr

# AI
npm install @anthropic-ai/sdk

# スクレイピング・RSS
npm install cheerio rss-parser robots-parser

# ユーティリティ
npm install date-fns zod string-similarity

# UI
npm install lucide-react

# 開発用
npm install -D @types/string-similarity
```

> **v1.1追加**: `robots-parser` を追加（robots.txt 対応）

**成果物**: package.json に依存関係追加済み
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 1（技術スタック）

---

### [T-003] Supabase プロジェクト作成

**ゴール**: データベース環境を構築

**手順**:
1. https://supabase.com でプロジェクト作成
2. プロジェクト名: `ai-knowledge-hub`
3. リージョン: Northeast Asia (Tokyo)
4. Project URL と API Keys を取得

**成果物**: Supabase プロジェクト稼働中
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 2（データベース設計）

---

### [T-004] 環境変数の設定

**ゴール**: API キーを安全に管理

**ファイル**: `.env.local`

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJxxx
SUPABASE_SERVICE_ROLE_KEY=eyJxxx

# Anthropic
ANTHROPIC_API_KEY=sk-ant-xxx

# API保護（v1.1追加）
API_SECRET_KEY=your-secret-key-here
```

**ファイル**: `.env.example`（Git にコミット）

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
ANTHROPIC_API_KEY=
API_SECRET_KEY=
```

**成果物**: 環境変数設定完了、`.gitignore` に `.env.local` 追加
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 1（環境変数）

---

### [T-005] ディレクトリ構成の作成

**ゴール**: プロジェクト構造を整備

```
src/
├── app/
│   ├── (dashboard)/
│   │   ├── page.tsx              # ダッシュボード
│   │   ├── articles/
│   │   │   ├── page.tsx          # 記事一覧
│   │   │   └── [id]/page.tsx     # 記事詳細
│   │   ├── sources/
│   │   │   └── page.tsx          # ソース管理
│   │   └── reports/
│   │       └── page.tsx          # レポート一覧
│   ├── api/
│   │   ├── sources/
│   │   │   └── route.ts
│   │   ├── articles/
│   │   │   ├── route.ts
│   │   │   └── [id]/
│   │   │       ├── route.ts
│   │   │       ├── review/route.ts
│   │   │       └── reanalyze/route.ts
│   │   ├── collect/
│   │   │   └── route.ts
│   │   └── reports/
│   │       ├── route.ts
│   │       └── generate/route.ts
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/                       # 共通UIコンポーネント
│   ├── articles/                 # 記事関連
│   ├── sources/                  # ソース関連
│   └── layout/                   # レイアウト
├── lib/
│   ├── supabase/
│   │   ├── client.ts             # ブラウザ用
│   │   └── server.ts             # サーバー用
│   ├── anthropic.ts              # Claude API
│   ├── collector/
│   │   ├── rss.ts                # RSS収集
│   │   ├── scraper.ts            # スクレイピング
│   │   └── robots.ts             # robots.txt チェック（v1.1追加）
│   ├── analyzer/
│   │   ├── filter.ts             # 事前フィルタ（v1.1追加）
│   │   └── prompt.ts             # 分析プロンプト
│   └── utils/
│       ├── url.ts                # URL正規化
│       └── similarity.ts         # タイトル類似度
├── middleware.ts                 # API保護（v1.1追加）
└── types/
    └── index.ts                  # 型定義
```

**成果物**: ディレクトリ構造作成完了
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 1（ディレクトリ構成）

---

## Phase 1: 基盤実装

### [T-006] Supabase クライアント設定

**ゴール**: Supabase接続の基盤を構築

**ファイル**: `src/lib/supabase/client.ts`

```typescript
import { createBrowserClient } from '@supabase/ssr';

export const createClient = () =>
  createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
```

**ファイル**: `src/lib/supabase/server.ts`

```typescript
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export const createClient = () => {
  const cookieStore = cookies();
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value;
        },
      },
    }
  );
};
```

**成果物**: Supabase クライアント設定完了
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 2.1（Supabase設定）

---

### [T-007] データベーステーブル作成

**ゴール**: PRD_02_Technical.md のスキーマを実装

**手順**: Supabase Dashboard → SQL Editor で実行

**SQL**: PRD_02_Technical.md Section 2.2 のSQL（4テーブル）

**成果物**: sources, articles, manuals, reports テーブル作成完了
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 2.2（テーブル定義SQL）

---

### [T-008] 型定義の作成

**ゴール**: TypeScript型でデータ構造を定義

**ファイル**: `src/types/index.ts`

```typescript
// Database types
export interface Source {
  id: string;
  name: string;
  url: string;
  feed_url: string | null;
  type: 'rss' | 'scrape';
  is_active: boolean;
  scrape_config: ScrapeConfig | null;
  created_at: string;
  updated_at: string;
}

export interface ScrapeConfig {
  titleSelector: string;
  contentSelector: string;
  dateSelector?: string;
  linkSelector?: string;
}

export interface Article {
  id: string;
  source_id: string;
  url: string;
  title: string;
  content: string | null;
  summary: string | null;
  published_at: string | null;
  collected_at: string;
  ai_importance: number | null;
  ai_usefulness: number | null;
  ai_difficulty: number | null;
  ai_reason: AIReason | null;
  ai_tags: string[] | null;
  ai_analyzed_at: string | null;
  ai_error: string | null;
  human_importance: number | null;
  human_usefulness: number | null;
  human_difficulty: number | null;
  human_comment: string | null;
  reviewed_at: string | null;
  status: 'pending' | 'approved' | 'rejected' | 'analysis_failed';
  created_at: string;
  updated_at: string;
}

// ... 以下続く
```

**成果物**: 全テーブルの型定義完了
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 2.2（テーブル定義）

---

### [T-009] API レスポンス型の定義

**ゴール**: PRD_02_Technical.md Section 3.2 の型を実装

**ファイル**: `src/types/api.ts`

```typescript
export interface ActionResponse<T = unknown> {
  success: boolean;
  message: string;
  data?: T;
  error?: {
    code: string;
    details?: string;
  };
}

export interface PaginatedResponse<T> {
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
```

**成果物**: APIレスポンス型定義完了
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.2（APIレスポンス形式）

---

### [T-009.1] API保護ミドルウェア実装 ⚡NEW

**ゴール**: 外部からのAPI不正アクセスを防止

**背景**: Vercel公開時、APIエンドポイントが誰でもアクセス可能になるリスク

**ファイル**: `src/middleware.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  // API Routes のみ保護
  if (request.nextUrl.pathname.startsWith('/api/')) {
    const apiKey = request.headers.get('x-api-key');
    const expectedKey = process.env.API_SECRET_KEY;

    // 開発環境ではスキップ
    if (process.env.NODE_ENV === 'development') {
      return NextResponse.next();
    }

    // 本番環境ではAPIキーチェック
    if (!apiKey || apiKey !== expectedKey) {
      return NextResponse.json(
        { success: false, message: '認証が必要です', error: { code: 'UNAUTHORIZED' } },
        { status: 401 }
      );
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: '/api/:path*',
};
```

**成果物**: APIエンドポイント保護完了
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3（API設計）

---

### [T-010] 共通UIコンポーネント作成

**ゴール**: 再利用可能なUIパーツを作成

**コンポーネント一覧**:

| コンポーネント | 用途 |
|---------------|------|
| `Button` | ボタン |
| `Card` | カード |
| `Badge` | バッジ（ステータス表示） |
| `Input` | 入力フィールド |
| `Select` | セレクトボックス |
| `Textarea` | テキストエリア |
| `StarRating` | 評価表示（★） |
| `ScoreInput` | スコア入力（1-5） |
| `Modal` | モーダルダイアログ |
| `Toast` | 通知メッセージ |

**成果物**: `src/components/ui/` に各コンポーネント
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4（画面設計）

---

### [T-011] レイアウトコンポーネント作成

**ゴール**: アプリ全体のレイアウトを構築

**ファイル**: `src/app/layout.tsx`

**コンポーネント**:
- `Header`: ヘッダー（タイトル、ナビゲーション）
- `Sidebar`: サイドバー（メニュー）
- `MainContent`: メインコンテンツエリア

**成果物**: 一貫したレイアウト構造
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4（画面設計）

---

### [T-012] ナビゲーション設定

**ゴール**: ページ間の遷移を設定

**メニュー構成**:

```typescript
const navigation = [
  { name: 'ダッシュボード', href: '/', icon: Home },
  { name: '記事一覧', href: '/articles', icon: FileText },
  { name: 'ソース管理', href: '/sources', icon: Globe },
  { name: 'レポート', href: '/reports', icon: BarChart },
];
```

**成果物**: ナビゲーションコンポーネント
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4.1（画面一覧）

---

### [T-013] エラーハンドリング基盤

**ゴール**: 共通のエラー処理を構築

**ファイル**: `src/lib/errors.ts`

```typescript
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
  }
}

export const handleApiError = (error: unknown): ActionResponse => {
  if (error instanceof AppError) {
    return {
      success: false,
      message: error.message,
      error: { code: error.code },
    };
  }
  return {
    success: false,
    message: '予期しないエラーが発生しました',
    error: { code: 'UNKNOWN_ERROR' },
  };
};
```

**成果物**: エラーハンドリングユーティリティ
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.2（エラーレスポンス形式）

---

## Phase 2: 収集機能

### [T-014] Sources API 実装

**ゴール**: 監視対象サイトのCRUD API

**エンドポイント**:

| Method | Path | 機能 |
|--------|------|------|
| GET | `/api/sources` | 一覧取得 |
| POST | `/api/sources` | 新規作成 |
| PUT | `/api/sources/[id]` | 更新 |
| DELETE | `/api/sources/[id]` | 削除 |

**成果物**: Sources API 完成
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.1（Sources API）

---

### [T-015] RSS パーサー実装

**ゴール**: RSSフィードから記事を取得

**ファイル**: `src/lib/collector/rss.ts`

```typescript
import Parser from 'rss-parser';
import * as cheerio from 'cheerio';

const parser = new Parser();

export const fetchRssArticles = async (feedUrl: string) => {
  const feed = await parser.parseURL(feedUrl);

  return Promise.all(feed.items.map(async item => {
    // RSSのスニペットが短い場合、元記事から本文取得を試みる
    let content = item.contentSnippet || item.content || '';

    if (content.length < 500 && item.link) {
      try {
        const fullContent = await fetchArticleContent(item.link);
        if (fullContent) {
          content = fullContent;
        }
      } catch {
        // 取得失敗時はスニペットを使用
      }
    }

    return {
      title: item.title || '',
      url: item.link || '',
      content: content.slice(0, 5000),
      publishedAt: item.pubDate ? new Date(item.pubDate) : null,
    };
  }));
};

// 元記事から本文を取得（汎用セレクタで試行）
const fetchArticleContent = async (url: string): Promise<string | null> => {
  const response = await fetch(url, {
    headers: { 'User-Agent': 'AI-Knowledge-Hub/1.0 (Compatible)' }
  });
  const html = await response.text();
  const $ = cheerio.load(html);

  // 一般的な記事本文セレクタを試行
  const selectors = [
    'article', '.article-content', '.post-content',
    '.entry-content', 'main article', '.markdown-body'
  ];

  for (const selector of selectors) {
    const text = $(selector).text().trim();
    if (text.length > 500) {
      return text.slice(0, 5000);
    }
  }

  return null;
};
```

> **v1.1強化**: RSSスニペットが短い場合、元記事から本文を取得する機能を追加

**成果物**: RSS取得機能
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 5（データ収集処理）

---

### [T-016] スクレイパー実装

**ゴール**: Cheerioでウェブページをパース（robots.txt対応）

**ファイル**: `src/lib/collector/scraper.ts`

```typescript
import * as cheerio from 'cheerio';
import { isAllowedByRobots } from './robots';
import { ScrapeConfig } from '@/types';

export const scrapeArticles = async (
  baseUrl: string,
  config: ScrapeConfig
): Promise<Article[]> => {
  // robots.txt チェック
  const allowed = await isAllowedByRobots(baseUrl);
  if (!allowed) {
    console.warn(`Skipped: ${baseUrl} (blocked by robots.txt)`);
    return [];
  }

  const response = await fetch(baseUrl, {
    headers: {
      'User-Agent': 'AI-Knowledge-Hub/1.0 (Compatible; +https://github.com/yourrepo)'
    }
  });

  if (!response.ok) {
    throw new Error(`Failed to fetch: ${response.status}`);
  }

  const html = await response.text();
  const $ = cheerio.load(html);

  // 設定されたセレクタで記事を抽出
  const articles = $(config.linkSelector || 'a[href*="/article"], a[href*="/post"]')
    .map((_, el) => {
      const $el = $(el);
      return {
        title: $el.find(config.titleSelector).text().trim() || $el.text().trim(),
        url: new URL($el.attr('href') || '', baseUrl).href,
        content: '', // 個別ページから取得
        publishedAt: null,
      };
    })
    .get()
    .filter(a => a.title && a.url);

  return articles;
};
```

**ファイル**: `src/lib/collector/robots.ts` ⚡NEW

```typescript
import robotsParser from 'robots-parser';

const robotsCache = new Map<string, { allowed: boolean; expires: number }>();

export const isAllowedByRobots = async (url: string): Promise<boolean> => {
  const { origin } = new URL(url);
  const robotsUrl = `${origin}/robots.txt`;

  // キャッシュチェック（1時間有効）
  const cached = robotsCache.get(origin);
  if (cached && cached.expires > Date.now()) {
    return cached.allowed;
  }

  try {
    const response = await fetch(robotsUrl);
    if (!response.ok) {
      // robots.txt がない場合は許可
      return true;
    }

    const robotsTxt = await response.text();
    const robots = robotsParser(robotsUrl, robotsTxt);
    const allowed = robots.isAllowed(url, 'AI-Knowledge-Hub') ?? true;

    // キャッシュに保存
    robotsCache.set(origin, {
      allowed,
      expires: Date.now() + 60 * 60 * 1000, // 1時間
    });

    return allowed;
  } catch {
    return true; // エラー時は許可
  }
};
```

> **v1.1追加**: robots.txt チェック機能、User-Agent設定

**成果物**: スクレイピング機能（robots.txt対応）
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 5（データ収集処理）

---

### [T-017] URL正規化・重複検出実装

**ゴール**: PRD_02_Technical.md Section 7 を実装

**ファイル**: `src/lib/utils/url.ts`, `src/lib/utils/similarity.ts`

**成果物**: 重複検出機能
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 7（重複検出）

---

### [T-018] 収集ジョブ実装

**ゴール**: 全ソースから記事を収集するジョブ

**ファイル**: `src/lib/collector/job.ts`

**フロー**:
1. アクティブなソースを取得
2. 各ソースからRSS/スクレイピングで記事取得
3. 重複チェック
4. 新規記事をDBに保存

**成果物**: 収集ジョブ関数
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 5（データ収集処理）

---

### [T-018.1] 事前フィルタ実装 ⚡NEW

**ゴール**: AI分析前に低品質な記事を除外しコストを削減

**背景**: 大量の記事を無条件でClaude APIに送るとコストが急増

**ファイル**: `src/lib/analyzer/filter.ts`

```typescript
interface FilterResult {
  passed: boolean;
  reason?: string;
}

// AI分析前のフィルタ
export const preAnalysisFilter = (article: {
  title: string;
  content: string;
  url: string;
}): FilterResult => {

  // 1. コンテンツが短すぎる
  if (!article.content || article.content.length < 200) {
    return { passed: false, reason: 'コンテンツが短すぎます（200文字未満）' };
  }

  // 2. タイトルがスパム的
  const spamPatterns = [
    /^pr\s*:/i,           // PR記事
    /広告|sponsored|ad/i,  // 広告
    /\[?募集\]?/,          // 求人
    /お知らせ|プレスリリース/,
  ];
  if (spamPatterns.some(p => p.test(article.title))) {
    return { passed: false, reason: 'スパム/広告記事の可能性' };
  }

  // 3. 関連性チェック（キーワード）
  const relevantKeywords = [
    'claude', 'ai', 'llm', 'gpt', 'コーディング', 'coding',
    'プログラミング', 'developer', '開発', 'api', 'prompt'
  ];
  const text = (article.title + ' ' + article.content).toLowerCase();
  const hasRelevantKeyword = relevantKeywords.some(kw => text.includes(kw));

  if (!hasRelevantKeyword) {
    return { passed: false, reason: '関連キーワードが含まれていません' };
  }

  return { passed: true };
};

// 1日あたりのAI分析上限
export const DAILY_ANALYSIS_LIMIT = 50;
```

**成果物**: 事前フィルタ機能（コスト制御）
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 6（AI分析処理）

---

### [T-019] Collect API 実装

**ゴール**: 手動収集実行エンドポイント

**エンドポイント**: `POST /api/collect`

**成果物**: 収集トリガーAPI
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.1（Collect API）

---

## Phase 3: 分析機能

### [T-020] Claude API クライアント実装

**ゴール**: Anthropic SDKの初期化とラッパー

**ファイル**: `src/lib/anthropic.ts`

**成果物**: Claude APIクライアント
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 6（AI分析処理）

---

### [T-021] 分析プロンプト作成

**ゴール**: evaluation-criteria.md に基づくプロンプト

**ファイル**: `src/lib/analyzer/prompt.ts`

```typescript
export const buildAnalysisPrompt = (article: {
  title: string;
  content: string;
  url: string;
}): string => {
  return `あなたはAI/プログラミング技術の専門家です。以下の記事を評価してください。

## 記事情報
タイトル: ${article.title}
URL: ${article.url}

## 記事内容
${article.content.slice(0, 4000)}

## 評価基準

### 重要度 (1-5)
- 5: 必須知識。知らないと重大な問題が起きる（破壊的変更、セキュリティ脆弱性など）
- 4: 強く推奨。生産性に大きく影響する
- 3: 有用。知っていると便利
- 2: 参考程度。特定状況で役立つ
- 1: 教養レベル。知らなくても困らない

### 有用度 (1-5)
- 5: 即座に使える。コピペで動くコード例付き
- 4: 少し調整で使える
- 3: 理解すれば応用可能
- 2: 参考になるが直接使えない
- 1: 理論のみ。実践困難

### 難易度 (1-5)
- 1: 初心者OK。30分以内で理解可能
- 2: 基礎知識があれば1時間程度
- 3: 中級者向け。半日程度
- 4: 上級者向け。1日以上
- 5: エキスパート向け。深い専門知識必須

## 回答形式（JSON）
{
  "importance": 数値,
  "importance_reason": "理由（50文字以内）",
  "usefulness": 数値,
  "usefulness_reason": "理由（50文字以内）",
  "difficulty": 数値,
  "difficulty_reason": "理由（50文字以内）",
  "summary": "記事の要約（100文字以内）",
  "tags": ["タグ1", "タグ2", "タグ3"]
}

JSONのみを出力してください。`;
};
```

**成果物**: 分析プロンプトテンプレート
**参照仕様書**: [evaluation-criteria.md](evaluation-criteria.md)（評価基準定義）

---

### [T-022] 分析ジョブ実装（リトライ付き）

**ゴール**: PRD_02_Technical.md Section 5.1 を実装

**ファイル**: `src/lib/analyzer/job.ts`

**フロー**:
1. 未分析の記事を取得
2. **事前フィルタでスクリーニング（T-018.1）**
3. フィルタ通過した記事のみAI分析
4. リトライ処理（3回まで）
5. 結果をDBに保存

**成果物**: AI分析機能（リトライ・エラーハンドリング付き）
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 6.1（分析フロー）, [evaluation-criteria.md](evaluation-criteria.md)

---

### [T-022.1] 分析コスト監視 ⚡NEW

**ゴール**: API利用コストを可視化

**機能**:
- 日次/月次の分析件数をカウント
- 上限に近づいたら警告表示
- ダッシュボードに利用状況を表示

**成果物**: コスト監視機能
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 6（AI分析処理）

---

### [T-023] Reanalyze API 実装

**ゴール**: 分析失敗記事の再分析エンドポイント

**エンドポイント**: `POST /api/articles/[id]/reanalyze`

**成果物**: 再分析API
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.1（Reanalyze API）

---

## Phase 4: UI実装

### [T-024] ダッシュボードページ

**ゴール**: 統計サマリーと最近の記事を表示

**パス**: `/`

**表示内容**:
- 今週の収集件数
- 未レビュー件数
- 承認済み件数
- **今月のAI分析件数/上限（v1.1追加）**
- 最近の記事リスト（10件）
- [収集実行] ボタン

**成果物**: ダッシュボードUI
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4.2（ダッシュボード画面）

---

### [T-025] ソース管理ページ

**ゴール**: 監視対象サイトのCRUD画面

**パス**: `/sources`

**機能**:
- ソース一覧表示
- 新規追加モーダル
- 編集モーダル
- 削除確認
- 有効/無効トグル

**成果物**: ソース管理UI
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4.3（ソース管理画面）

---

### [T-026] 記事一覧ページ

**ゴール**: 収集した記事の一覧表示

**パス**: `/articles`

**機能**:
- 記事カード一覧
- ステータスフィルター（全て/未レビュー/承認済/却下/分析失敗）
- ソースフィルター
- ページネーション
- AI評価スコア表示

**成果物**: 記事一覧UI
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4.4（記事一覧画面）

---

### [T-027] 記事詳細・レビューページ

**ゴール**: 記事内容表示とレビュー入力

**パス**: `/articles/[id]`

**機能**:
- 記事タイトル・ソース・日時
- 元記事リンク
- AI要約表示
- AI評価スコア表示（理由付き）
- 人間評価入力（1-5 × 3項目）
- コメント入力
- 承認/却下ボタン
- [再分析] ボタン（分析失敗時）

**成果物**: レビューUI
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4.5（記事詳細・レビュー画面）

---

### [T-028] Review API 実装

**ゴール**: 人間レビュー保存エンドポイント

**エンドポイント**: `PUT /api/articles/[id]/review`

**成果物**: レビュー保存API
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.1（Review API）

---

### [T-029] レポート一覧ページ

**ゴール**: 週次レポートの一覧表示

**パス**: `/reports`

**機能**:
- レポート一覧（期間、記事数）
- [レポート生成] ボタン
- レポート詳細表示

**成果物**: レポート一覧UI
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4.6（レポート画面）

---

### [T-030] レポート生成ロジック

**ゴール**: 承認済み記事から週次レポートを生成

**ファイル**: `src/lib/report/generator.ts`

**内容**:
- 期間内の承認済み記事を集計
- カテゴリ（タグ）別に分類
- スコア順にランキング
- Markdown形式で出力

**成果物**: レポート生成機能
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 8（レポート生成）

---

### [T-031] Reports API 実装

**ゴール**: レポートCRUD API

**エンドポイント**:

| Method | Path | 機能 |
|--------|------|------|
| GET | `/api/reports` | 一覧取得 |
| POST | `/api/reports/generate` | 新規生成 |

**成果物**: Reports API
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.1（Reports API）

---

### [T-032] Articles API 実装

**ゴール**: 記事一覧・詳細API

**エンドポイント**:

| Method | Path | 機能 |
|--------|------|------|
| GET | `/api/articles` | 一覧取得（ページネーション） |
| GET | `/api/articles/[id]` | 詳細取得 |

**成果物**: Articles API
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 3.1（Articles API）

---

### [T-033] ローディング・エラー状態

**ゴール**: 各ページのローディングとエラー表示

**ファイル**: 各ルートの `loading.tsx`, `error.tsx`

**成果物**: UX改善（ローディング・エラー表示）
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 4（画面設計）

---

## Phase 5: 統合・テスト

### [T-034] 手動E2Eテスト

**ゴール**: 全機能の動作確認

**テストシナリオ**:

| # | シナリオ | 確認項目 |
|---|----------|----------|
| 1 | ソース追加 | RSS/スクレイプの登録 |
| 2 | 手動収集 | 記事がDBに保存される |
| 3 | AI分析 | スコア・要約が生成される |
| 4 | レビュー | 人間評価が保存される |
| 5 | レポート生成 | 週次レポートが作成される |
| 6 | **robots.txt拒否サイト** | スクレイピングがスキップされる |
| 7 | **事前フィルタ** | 低品質記事がAI分析をスキップ |

**成果物**: テスト完了チェックリスト
**参照仕様書**: [PRD_01_Overview.md](Specifications/PRD_01_Overview.md), [PRD_02_Technical.md](Specifications/PRD_02_Technical.md)（全体検証）

---

### [T-035] 初期データ投入

**ゴール**: デモ用の監視対象を登録

**データ**:

| サイト | タイプ | URL |
|--------|--------|-----|
| Zenn (Claude) | RSS | https://zenn.dev/topics/claude/feed |
| Qiita (AI) | RSS | https://qiita.com/tags/ai/feed |
| Anthropic Blog | Scrape | https://www.anthropic.com/news |

**成果物**: 初期ソースデータ
**参照仕様書**: [PRD_01_Overview.md](Specifications/PRD_01_Overview.md) Section 2.2（対象サイト例）

---

### [T-036] Vercel デプロイ

**ゴール**: 本番環境にデプロイ

**手順**:
1. GitHub リポジトリ作成・プッシュ
2. Vercel にインポート
3. 環境変数設定（**API_SECRET_KEY含む**）
4. デプロイ

**成果物**: 本番URL
**参照仕様書**: [PRD_02_Technical.md](Specifications/PRD_02_Technical.md) Section 1（環境変数）

---

### [T-037] README.md 作成

**ゴール**: プロジェクトドキュメント

**内容**:
- プロジェクト概要
- セットアップ手順
- 環境変数一覧
- 開発コマンド

**成果物**: README.md
**参照仕様書**: [PRD_01_Overview.md](Specifications/PRD_01_Overview.md)（プロジェクト概要）

---

## 📋 依存関係マップ

```
Phase 0 (環境構築)
  T-001 ──→ T-002 ──→ T-005
  T-003 ──→ T-004

Phase 1 (基盤実装)
  T-006 ──→ T-007 ──→ T-008 ──→ T-009 ──→ T-009.1
  T-010 ──→ T-011 ──→ T-012
  T-013

Phase 2 (収集機能)
  T-014 ──→ T-015 ──→ T-018 ──→ T-018.1 ──→ T-019
         ──→ T-016 ──┘
         ──→ T-017 ──┘

Phase 3 (分析機能)
  T-020 ──→ T-021 ──→ T-022 ──→ T-022.1 ──→ T-023

Phase 4 (UI実装)
  T-024 ──→ T-025 ──→ T-026 ──→ T-027
  T-028, T-029 ──→ T-030 ──→ T-031
  T-032, T-033

Phase 5 (統合・テスト)
  T-034 ──→ T-035 ──→ T-036 ──→ T-037
```

---

## ✅ 完了チェックリスト

- [ ] Phase 0: 環境構築 (5タスク)
- [ ] Phase 1: 基盤実装 (9タスク)
- [ ] Phase 2: 収集機能 (7タスク)
- [ ] Phase 3: 分析機能 (5タスク)
- [ ] Phase 4: UI実装 (10タスク)
- [ ] Phase 5: 統合・テスト (4タスク)

---

## 📝 将来の拡張（Phase 2以降）

> 外部レビューで提案された機能（MVP後に検討）

| 機能 | 説明 | 優先度 |
|------|------|--------|
| Supabase Auth | ユーザー認証（社内公開用） | HIGH |
| Markdownプレビュー | レポートのリッチ表示 + コピー機能 | MEDIUM |
| 「あとで読む」フラグ | 承認/却下とは別のブックマーク機能 | MEDIUM |
| タグベース関連マニュアル | マニュアル相互リンクの自動推薦 | LOW |
| Cron自動化 | GitHub Actions または外部サービス連携 | LOW |

---

## 📝 備考

- 各タスクの完了時に TASKS.md を更新
- 仕様変更が発生した場合は PRD を先に更新
- 不明点は実装前に確認
- **v1.1**: 外部レビュー反映（API保護、robots.txt、コスト制御）
