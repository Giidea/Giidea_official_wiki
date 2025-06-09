# **公式Wikiコンテンツ構造 ガイドライン for AI Assistant**

## **1. 基本方針**

公式Wikiのコンテンツ管理は、**「フラットなフォルダ構造」と、ファイル内の「Frontmatterメタデータ」**を組み合わせた方式を採用する。物理的なフォルダ分けは行わず、全ての分類と関係性はメタデータによって定義する。これにより、最大限の柔軟性と拡張性を確保する。

## **2. ディレクトリ構造とファイル命名**

全てのWikiコンテンツは、リポジトリのルートにある `content/` ディレクトリ内に、階層を作らずフラットに格納する。

Markdownファイルのファイル名は、URLの一部として利用されることを想定し、英数字の小文字とハイフン (-) のみで構成する。

```
.
└── content/
    ├── react-hooks-basics.md
    ├── state-management.md
    └── nodejs-event-loop.md
```

## **3. Markdownファイルの内部構造ルール (Frontmatter)**

全てのMarkdownファイルは、ファイルの先頭に必ずYAML Frontmatterを持つ。

### **3.1 基本メタデータ**

| フィールド名 | データ型 | 説明 |
|-------------|----------|------|
| title | string | 記事の正式なタイトル。 |
| slug | string | URLに使われる一意なID。全てのコンテンツ間で重複してはならない。 |
| tags | string[] | コンテンツのカテゴリや関連技術を示すタグの配列。 |
| author_id | string | 初版を作成したユーザーのID（UUID）。 |
| created_at | string | 初版が作成された日時（ISO 8601形式）。 |

### **3.2 【重要】セマンティック関係メタデータ (relations)**

コンテンツ間の意味的な繋がりを定義するための、このプラットフォームの核となるメタデータ。

**フィールド名**: `relations`

**データ型**: object[] （オブジェクトの配列）

**オブジェクトの構造**:
- `type` (string): 関係性の種類 (parent, child, prerequisite, alternative, comparison)
- `slug` (string): 関連するコンテンツのslug

| type | 意味 | UIでの表示例 |
|------|------|-------------|
| parent | 親トピック（上位概念） | パンくずリスト、上位トピック |
| child | 子トピック（下位概念） | サイドバーの目次 |
| prerequisite | 前提知識（これを読む前に） | 「前提知識」セクション |
| alternative | 代替技術（目的は似ているがアプローチが違う） | 「代替技術」セクション |
| comparison | 比較対象（直接的な競合・よく比較される） | 「比較と検討」セクション |

## **4. Frontmatter記述例**

### **ファイル**: `content/react.md`

```yaml
---
title: "React"
slug: "react"
tags: ["JavaScript", "Frontend", "Library"]
relations:
  - type: "alternative"
    slug: "vue"
  - type: "alternative"
    slug: "svelte"
  - type: "comparison"
    slug: "angular"
---

# React
...
```

### **ファイル**: `content/state-management-libraries.md`

```yaml
---
title: "状態管理ライブラリ"
slug: "state-management-libraries"
tags: ["状態管理", "React", "Vue"]
relations:
  - type: "parent"
    slug: "react"
  - type: "parent"
    slug: "vue"
  - type: "comparison"
    slug: "redux"
  - type: "comparison"
    slug: "zustand"
  - type: "comparison"
    slug: "pinia"
---

# 状態管理ライブラリ
...
```

## **5. アプリケーションでの利用方法**

AIは、この構造を以下のルールで解釈し、アプリケーションの機能を実装すること。

### **URLの生成**:
WikiページのURLは、`/wiki/[slug]` の形式で生成する。

### **動的なUI構築**:
ページ表示時、relationsメタデータを解析する。

- `type: "child"` の関係を持つページへのリンクを、ナビゲーション用のサイドバー（木構造の子要素）として表示する。
- `type: "parent"` の関係を持つページへのリンクを、「上位のトピック」としてパンくずリストなどに表示する。
- `type: "alternative"` や `type: "comparison"` の関係を持つページへのリンクを、ページ下部の「代替技術や比較」セクションにまとめて表示する。