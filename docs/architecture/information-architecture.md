# FAQシステム 情報アーキテクチャ設計書

## 1. 📍 サイトマップ構造

```mermaid
graph TD
    subgraph "公開エリア"
        A[トップページ]
        B[FAQ一覧/検索]
        C[FAQ詳細]
        D[問合せ]
        E[ユーザー認証]
    end

    subgraph "会員エリア"
        F[マイページ]
        G[問合せ履歴]
        H[お気に入りFAQ]
    end

    subgraph "管理エリア"
        I[管理ダッシュボード]
        J[FAQ管理]
        K[カテゴリ/タグ管理]
        L[問合せ管理]
        M[ユーザー管理]
        N[統計/レポート]
    end

    A --> B
    B --> C
    C --> D
    A --> E
    E --> F
    F --> G
    F --> H
    E --> I
    I --> J
    I --> K
    I --> L
    I --> M
    I --> N
```

## 2. 🔄 コンテンツ階層構造

```mermaid
graph TD
    subgraph "FAQコンテンツ"
        A[カテゴリ]
        B[サブカテゴリ]
        C[FAQ記事]
        D[タグ]
        E[関連FAQ]
    end

    subgraph "ユーザーコンテンツ"
        F[問合せ]
        G[コメント]
        H[評価]
    end

    subgraph "管理コンテンツ"
        I[管理者メモ]
        J[テンプレート]
        K[統計データ]
    end

    A --> B
    B --> C
    C --> D
    C --> E
    C --> F
    F --> G
    C --> H
    F --> I
    F --> J
    A --> K
```

## 3. 🧩 コンポーネント階層設計

```mermaid
graph TD
    subgraph "共通コンポーネント"
        A[ヘッダー]
        B[フッター]
        C[サイドナビ]
        D[パンくず]
        E[検索バー]
    end

    subgraph "機能コンポーネント"
        F[FAQリスト]
        G[FAQビューワー]
        H[問合せフォーム]
        I[フィルター]
        J[ページネーション]
    end

    subgraph "管理コンポーネント"
        K[エディター]
        L[ダッシュボード]
        M[データテーブル]
        N[チャート]
    end

    A --> E
    F --> I
    F --> J
    G --> H
    L --> N
    L --> M
```

## 4. 🔍 データ関係モデル

```mermaid
erDiagram
    CATEGORY ||--o{ FAQ : contains
    FAQ ||--o{ TAG : has
    FAQ ||--o{ COMMENT : has
    FAQ ||--o{ RATING : has
    FAQ ||--o{ RELATED_FAQ : has
    USER ||--o{ INQUIRY : creates
    USER ||--o{ COMMENT : posts
    USER ||--o{ RATING : gives
    INQUIRY ||--o{ INQUIRY_STATUS : has
    INQUIRY ||--o{ TEMPLATE : uses
```

## 5. 🚶 主要ユーザーフロー

### 5.1 FAQ検索・閲覧フロー
```mermaid
sequenceDiagram
    actor User
    participant Search
    participant FAQ
    participant Related
    
    User->>Search: キーワード入力
    Search->>FAQ: 検索実行
    FAQ->>User: 検索結果表示
    User->>FAQ: FAQ選択
    FAQ->>Related: 関連FAQ取得
    Related->>User: 関連コンテンツ表示
```

### 5.2 問合せ作成フロー
```mermaid
sequenceDiagram
    actor User
    participant Form
    participant Validation
    participant Suggestion
    participant System
    
    User->>Form: フォーム開始
    Form->>Suggestion: 類似FAQ検索
    Suggestion->>User: FAQ提案表示
    User->>Form: 内容入力
    Form->>Validation: 入力検証
    Validation->>User: フィードバック
    User->>Form: 送信
    Form->>System: 保存処理
```

## 6. 🎯 SEO最適化設計

### メタデータ構造
```yaml
# ページ共通メタデータ
base_meta:
  charset: UTF-8
  viewport: width=device-width, initial-scale=1
  robots: index, follow

# FAQ記事メタデータ
faq_meta:
  title: {カテゴリ} - {FAQ表題} | {サイト名}
  description: {FAQ要約, 最大160文字}
  keywords: {関連キーワード, カンマ区切り}
  og:type: article
  article:published_time: {公開日時}
  article:modified_time: {更新日時}
```

### 構造化データ
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "FAQ質問",
    "acceptedAnswer": {
      "@type": "Answer",
      "text": "FAQ回答"
    }
  }]
}
```

## 7. ♿ アクセシビリティ設計

### WAI-ARIA実装方針
```yaml
# ランドマーク
landmarks:
  header: role="banner"
  nav: role="navigation"
  main: role="main"
  search: role="search"
  footer: role="contentinfo"

# コンポーネント
components:
  search:
    input: aria-label="検索"
    button: aria-label="検索実行"
  
  faq_list:
    region: role="region" aria-label="FAQ一覧"
    item: role="article"
  
  pagination:
    nav: role="navigation" aria-label="ページネーション"
    button: aria-label="ページ{n}へ移動"
```

## 8. 📱 レスポンシブ対応方針

### ブレークポイント定義
```css
// デバイス別ブレークポイント
@mixin mobile {
  @media (max-width: 767px) { @content; }
}

@mixin tablet {
  @media (min-width: 768px) and (max-width: 1023px) { @content; }
}

@mixin desktop {
  @media (min-width: 1024px) { @content; }
}

// レイアウトパターン
.layout {
  // モバイル: 縦積みレイアウト
  @include mobile {
    flex-direction: column;
  }
  
  // タブレット: 2カラムグリッド
  @include tablet {
    grid-template-columns: repeat(2, 1fr);
  }
  
  // デスクトップ: 3カラムグリッド
  @include desktop {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

## 9. 🔒 状態管理パターン

```mermaid
graph TD
    subgraph "グローバル状態"
        A[認証状態]
        B[ユーザー設定]
        C[システム通知]
    end

    subgraph "ページ状態"
        D[検索条件]
        E[フィルター設定]
        F[ソート設定]
    end

    subgraph "コンポーネント状態"
        G[フォーム入力]
        H[UI状態]
        I[アニメーション]
    end

    A --> D
    B --> E
    D --> G
    E --> H