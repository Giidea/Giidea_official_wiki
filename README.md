# Giidea Official Wiki

## 概要

Giidea Official Wikiは、開発者向けの技術知識ベースです。ITwikiartcileプラットフォームの実装パターンに基づいて構築されており、フラットなファイル構造とFrontmatterメタデータによる分類システムを採用しています。

## 構造

```
Giidea_official_wiki/
├── content/              # 全てのWikiコンテンツ（フラット構造）
│   ├── react-hooks-basics.md
│   ├── state-management.md
│   └── nodejs-event-loop.md
├── styleguide.md         # コンテンツ構造ガイドライン
└── README.md            # このファイル
```

## 設計原則

### 1. フラットなフォルダ構造
- 全てのMarkdownファイルは`content/`ディレクトリに配置
- カテゴリごとのサブディレクトリは作成しない
- 分類は Frontmatter メタデータで管理

### 2. Frontmatter メタデータ
各Markdownファイルには以下のメタデータを含める：

```yaml
---
title: "記事のタイトル"
slug: "url-friendly-identifier"
tags: ["tag1", "tag2", "tag3"]
author_id: "作成者ID"
created_at: "2024-01-15T09:00:00Z"
updated_at: "2024-01-15T09:00:00Z"
category: "カテゴリ"
difficulty: "beginner|intermediate|advanced"
estimated_reading_time: 10
---
```

### 3. ファイル命名規則
- 英数字の小文字とハイフン（-）のみ使用
- URL の一部として利用されることを考慮
- 例: `react-hooks-basics.md`, `nodejs-event-loop.md`

## 現在のコンテンツ

### Frontend
- [React Hooks の基礎](./content/react-hooks-basics.md) - React Hooksの基本的な使い方
- [State Management パターン](./content/state-management.md) - 各種状態管理ライブラリの比較

### Backend  
- [Node.js Event Loop の仕組み](./content/nodejs-event-loop.md) - 非同期処理の内部動作

## 貢献方法

1. **新しい記事の追加**
   - `content/` ディレクトリに新しいMarkdownファイルを作成
   - 適切な Frontmatter メタデータを含める
   - [styleguide.md](./styleguide.md) の規則に従う

2. **既存記事の改善**
   - GitHub PR形式での改善提案
   - 内容の更新時は `updated_at` フィールドを更新

3. **品質基準**
   - 技術的正確性の確保
   - 実際のコード例を含める
   - 適切な難易度設定
   - 関連記事へのリンク

## ITwikiarticle連携

このWikiは[ITwikiartcile](../ITwikiarticle/)プラットフォームと連携して動作し、以下の機能を提供します：

- **パーソナルフォーク**: 個人学習用にコンテンツをカスタマイズ
- **改善提案**: GitHub PR経由での内容改善
- **学習トラッキング**: 個人の学習進捗管理
- **タグベース検索**: メタデータによる柔軟な検索

## 技術仕様

- **Markdown形式**: 標準的なMarkdown記法
- **Frontmatter**: YAML形式のメタデータ
- **バージョン管理**: Git履歴による変更追跡
- **統合プラットフォーム**: ITwikiartcile Next.js アプリケーション

---

詳細は [styleguide.md](./styleguide.md) を参照してください。
