# FAQシステム API仕様書

## 1. 概要 📋

このドキュメントではFAQシステムのREST APIインターフェースについて定義します。

### ベースURL
```
https://api.faq-system.example.com/v1
```

### 共通ヘッダー
```
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
```

## 2. 認証・認可 🔒

### トークン取得
```http
POST /auth/token
```

#### リクエスト
```json
{
  "email": "string",
  "password": "string"
}
```

#### レスポンス
```json
{
  "access_token": "string",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## 3. FAQ API 📚

### FAQ一覧取得
```http
GET /faqs
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|------------|-----|------|------|
| page | integer | No | ページ番号（デフォルト: 1） |
| per_page | integer | No | 1ページの件数（デフォルト: 20, 最大: 100） |
| category_id | integer | No | カテゴリID |
| tag_id | integer | No | タグID |
| q | string | No | 検索キーワード |
| status | integer | No | ステータス（0:下書き, 1:公開, 2:非公開） |

#### レスポンス
```json
{
  "total": 100,
  "page": 1,
  "per_page": 20,
  "faqs": [
    {
      "id": 1,
      "title": "string",
      "content": "string",
      "categories": [
        {
          "id": 1,
          "name": "string"
        }
      ],
      "tags": [
        {
          "id": 1,
          "name": "string"
        }
      ],
      "view_count": 0,
      "helpful_rate": 0.95,
      "created_at": "2025-03-23T12:00:00Z",
      "updated_at": "2025-03-23T12:00:00Z"
    }
  ]
}
```

### FAQ詳細取得
```http
GET /faqs/{id}
```

#### レスポンス
```json
{
  "id": 1,
  "title": "string",
  "content": "string",
  "categories": [
    {
      "id": 1,
      "name": "string"
    }
  ],
  "tags": [
    {
      "id": 1,
      "name": "string"
    }
  ],
  "attachments": [
    {
      "id": 1,
      "filename": "string",
      "mime_type": "string",
      "url": "string"
    }
  ],
  "view_count": 0,
  "helpful_rate": 0.95,
  "created_at": "2025-03-23T12:00:00Z",
  "updated_at": "2025-03-23T12:00:00Z"
}
```

### FAQ作成
```http
POST /faqs
```

#### リクエスト
```json
{
  "title": "string",
  "content": "string",
  "category_ids": [1, 2],
  "tag_ids": [1, 2],
  "status": 0
}
```

#### レスポンス
```json
{
  "id": 1,
  "message": "FAQ created successfully"
}
```

### FAQ更新
```http
PUT /faqs/{id}
```

#### リクエスト
```json
{
  "title": "string",
  "content": "string",
  "category_ids": [1, 2],
  "tag_ids": [1, 2],
  "status": 1
}
```

#### レスポンス
```json
{
  "message": "FAQ updated successfully"
}
```

### FAQ削除
```http
DELETE /faqs/{id}
```

#### レスポンス
```json
{
  "message": "FAQ deleted successfully"
}
```

## 4. カテゴリ API 📂

### カテゴリ一覧取得
```http
GET /categories
```

#### レスポンス
```json
{
  "categories": [
    {
      "id": 1,
      "name": "string",
      "parent_id": null,
      "display_order": 1,
      "children": [
        {
          "id": 2,
          "name": "string",
          "parent_id": 1,
          "display_order": 1
        }
      ]
    }
  ]
}
```

### カテゴリ作成
```http
POST /categories
```

#### リクエスト
```json
{
  "name": "string",
  "parent_id": null,
  "display_order": 1
}
```

## 5. タグ API 🏷️

### タグ一覧取得
```http
GET /tags
```

#### レスポンス
```json
{
  "tags": [
    {
      "id": 1,
      "name": "string",
      "usage_count": 10
    }
  ]
}
```

## 6. フィードバック API 📝

### フィードバック送信
```http
POST /faqs/{id}/feedback
```

#### リクエスト
```json
{
  "rating": 5,
  "comment": "string"
}
```

## 7. 問合せ API ✉️

### 問合せ作成
```http
POST /inquiries
```

#### リクエスト
```json
{
  "subject": "string",
  "content": "string",
  "attachments": [
    {
      "filename": "string",
      "content_base64": "string"
    }
  ]
}
```

### 問合せ一覧取得
```http
GET /inquiries
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|------------|-----|------|------|
| page | integer | No | ページ番号（デフォルト: 1） |
| status | integer | No | ステータス（0:新規, 1:対応中, 2:解決済み） |

## 8. 統計 API 📊

### FAQ統計取得
```http
GET /stats/faqs
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|------------|-----|------|------|
| start_date | string | Yes | 開始日（YYYY-MM-DD） |
| end_date | string | Yes | 終了日（YYYY-MM-DD） |

#### レスポンス
```json
{
  "total_views": 1000,
  "total_feedbacks": 100,
  "average_rating": 4.5,
  "top_viewed": [
    {
      "id": 1,
      "title": "string",
      "view_count": 100
    }
  ],
  "top_rated": [
    {
      "id": 1,
      "title": "string",
      "rating": 4.8
    }
  ]
}
```

## 9. エラーレスポンス 🚫

### エラーレスポンス形式
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": {}
  }
}
```

### 共通エラーコード
| コード | 説明 |
|--------|------|
| 400 | リクエストパラメータが不正 |
| 401 | 認証エラー |
| 403 | 権限エラー |
| 404 | リソースが見つからない |
| 409 | リソースの競合 |
| 422 | バリデーションエラー |
| 429 | リクエスト制限超過 |
| 500 | サーバーエラー |

### レート制限
- 認証済みAPI: 1000リクエスト/時
- 未認証API: 100リクエスト/時

## 10. WebSocket API 🔄

### リアルタイム通知
```
ws://api.faq-system.example.com/v1/notifications
```

#### イベントタイプ
| イベント | 説明 |
|----------|------|
| faq.created | FAQ作成 |
| faq.updated | FAQ更新 |
| faq.deleted | FAQ削除 |
| inquiry.created | 問合せ作成 |
| inquiry.updated | 問合せ更新 |

#### メッセージフォーマット
```json
{
  "event": "string",
  "data": {}
}
```

## 11. APIバージョニング 🔖

- URLパスにバージョンを含める（例: `/v1/faqs`）
- 古いバージョンは6ヶ月間の移行期間後に廃止
- 新バージョンリリース時は事前通知を実施

## 12. セキュリティ要件 🔒

### 認証
- JWTベースの認証
- トークン有効期限: 1時間
- リフレッシュトークン有効期限: 30日

### 通信暗号化
- 全てのエンドポイントでTLS 1.3を使用
- 証明書: EV SSL

### アクセス制御
- RBACによる権限制御
- APIキーによるクライアント認証
- IPアドレスによるアクセス制限