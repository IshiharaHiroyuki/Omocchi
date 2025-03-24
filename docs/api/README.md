# FAQシステム API仕様書 📚

## 概要

このディレクトリには、FAQシステムのREST API仕様が含まれています。API仕様はOpenAPI (Swagger) 形式で記述されており、以下の機能を提供します：

- 認証・認可
- FAQ管理
- カテゴリ/タグ管理
- フィードバック管理
- 検索機能

## ファイル構成

```
docs/api/
├── README.md                    # 本ドキュメント
├── openapi/                     # OpenAPI仕様ファイル
│   ├── common.yaml             # 共通定義
│   ├── auth.yaml              # 認証・認可API
│   ├── faq.yaml               # FAQ管理API
│   ├── taxonomy.yaml          # カテゴリ/タグ管理API
│   ├── feedback.yaml          # フィードバックAPI
│   ├── search.yaml            # 検索API
│   └── schemas/               # 共通スキーマ定義
│       ├── responses.yaml     # レスポンス定義
│       ├── models.yaml        # モデル定義
│       └── security.yaml      # セキュリティ定義
└── examples/                   # APIリクエスト/レスポンス例
    ├── auth/
    ├── faq/
    ├── taxonomy/
    ├── feedback/
    └── search/
```

## 使用方法

### 開発環境での利用

1. APIドキュメントの表示
```bash
# Swagger UIを使用してドキュメントを表示
npm run docs:serve
```

2. OpenAPI仕様の検証
```bash
# OpenAPI仕様のバリデーション実行
npm run docs:validate
```

### APIクライアントの生成

```bash
# TypeScript用のAPIクライアント生成
npm run generate:client:typescript

# その他の言語用のクライアント生成
npm run generate:client -- --lang <language>
```

## 開発ガイドライン

### 1. API設計原則

- RESTfulな設計を採用
- リソース指向のURL設計
- 適切なHTTPメソッドの使用
- 一貫性のあるレスポンス形式
- バージョニングの明示

### 2. セキュリティ

- 認証にはJWTを使用
- 適切な認可制御の実施
- レート制限の適用
- 機密情報の適切な取り扱い

### 3. パフォーマンス

- 効果的なキャッシュ戦略
- ページネーションの実装
- 適切なインデックス設計
- レスポンスの最適化

### 4. エラー処理

- 標準的なエラーレスポンス形式
- 適切なHTTPステータスコードの使用
- エラーメッセージの多言語対応
- デバッグ情報の適切な制御

### 5. ドキュメント管理

- 変更履歴の記録
- 破壊的変更の明示
- サンプルコードの提供
- テストケースの文書化

## バージョン管理

- メジャーバージョン: URLパスで管理 (/v1/...)
- マイナーバージョン: OpenAPI仕様書のバージョンで管理
- 変更履歴はCHANGELOG.mdで管理

## 貢献ガイドライン

1. APIの変更を行う場合：
   - 仕様書の更新
   - テストケースの追加/更新
   - サンプルコードの更新
   - CHANGELOG.mdの更新

2. レビュープロセス：
   - OpenAPI仕様のバリデーション
   - バックエンド・フロントエンドチーム間のレビュー
   - ドキュメントの整合性確認
   - 破壊的変更の確認

## 問い合わせ先

- API仕様に関する質問: api-support@example.com
- バグ報告: https://github.com/your-org/your-repo/issues
- ドキュメント改善提案: プルリクエストを歓迎します

## ライセンス

本ドキュメントおよびAPI仕様は社内限りとし、許可なく外部への公開を禁じます。