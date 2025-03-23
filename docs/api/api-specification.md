# FAQシステム API仕様書

## 1. 概要 📋

このドキュメントではFAQシステムのREST APIインターフェースについて定義します。本仕様書は後にOpenAPI (Swagger) 形式に移行予定です。

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

### 共通レスポンスヘッダー
```
X-Total-Count: 総件数
X-Page-Number: 現在のページ番号
X-Page-Size: 1ページあたりの件数
Link: <https://api.example.com/v1/faqs?page=2>; rel="next",
      <https://api.example.com/v1/faqs?page=10>; rel="last"
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

### キャッシュ戦略
- GET リクエストに対してETタグとCache-Controlヘッダーを使用
- 公開FAQは1時間のキャッシュ
- 検索結果は5分のキャッシュ
- 管理系APIはキャッシュなし

### アクセス制御
システムは以下のロールを定義します：
- `admin`: システム管理者（全ての操作が可能）
- `content_manager`: コンテンツ管理者（FAQ作成・編集・削除が可能）
- `support_agent`: サポート担当者（FAQ閲覧、問合せ対応が可能）
- `user`: 一般ユーザー（FAQ閲覧、フィードバック送信が可能）

各APIエンドポイントで必要な権限を明示します。

## 2. 認証・認可 🔒

### トークン取得
```http
POST /auth/token
```

#### 必要な権限
- 認証不要

#### リクエストバリデーション
```json
{
  "email": {
    "type": "string",
    "format": "email",
    "required": true,
    "description": "有効なメールアドレス"
  },
  "password": {
    "type": "string",
    "minLength": 8,
    "maxLength": 100,
    "required": true,
    "description": "パスワード（8文字以上）"
  }
}
```

#### リクエスト例
```json
{
  "email": "user@example.com",
  "password": "your-secure-password"
}
```

#### レスポンス
```json
{
  "access_token": "string",
  "refresh_token": "string",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user": {
    "id": "string",
    "email": "string",
    "roles": ["user"],
    "permissions": ["read:faqs", "create:feedback"]
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "invalid_credentials",
    "message": "メールアドレスまたはパスワードが正しくありません",
    "details": {
      "attempts_remaining": 3,
      "lockout_time": "2025-03-23T12:30:00Z"
    }
  }
}
```

#### レート制限
- 同一IPアドレスからの試行: 10回/分
- 同一アカウントへの試行: 5回/分

### リフレッシュトークンの更新
```http
POST /auth/refresh
```

#### 必要な権限
- 有効なリフレッシュトークン

#### リクエスト
```json
{
  "refresh_token": "string"
}
```

#### レスポンス
```json
{
  "access_token": "string",
  "refresh_token": "string",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### トークン無効化（ログアウト）
```http
POST /auth/logout
```

#### 必要な権限
- 有効なアクセストークン

#### レスポンス
```json
{
  "message": "ログアウトしました"
}
```

## 3. FAQ API 📚

### FAQ一覧取得
```http
GET /faqs
```

#### 必要な権限
- 公開FAQの取得: 認証不要
- 非公開FAQの取得: `read:faqs`権限
- 下書きFAQの取得: `manage:faqs`権限

#### クエリパラメータバリデーション
```json
{
  "page": {
    "type": "integer",
    "minimum": 1,
    "default": 1,
    "description": "ページ番号"
  },
  "per_page": {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "default": 20,
    "description": "1ページの件数"
  },
  "category_id": {
    "type": "integer",
    "minimum": 1,
    "description": "カテゴリID"
  },
  "tag_id": {
    "type": "integer",
    "minimum": 1,
    "description": "タグID"
  },
  "q": {
    "type": "string",
    "maxLength": 100,
    "description": "検索キーワード"
  },
  "status": {
    "type": "integer",
    "enum": [0, 1, 2],
    "description": "ステータス（0:下書き, 1:公開, 2:非公開）"
  },
  "sort": {
    "type": "string",
    "enum": ["created_at", "updated_at", "view_count", "helpful_rate"],
    "default": "created_at",
    "description": "ソートフィールド"
  },
  "order": {
    "type": "string",
    "enum": ["asc", "desc"],
    "default": "desc",
    "description": "ソート順"
  }
}
```

#### キャッシュ戦略
- 公開FAQのみキャッシュ（1時間）
- 検索結果は短時間キャッシュ（5分）
- 認証済みユーザーの結果はキャッシュなし

#### レスポンスヘッダー
```
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
X-Total-Count: 100
X-Page-Number: 1
X-Page-Size: 20
Link: <https://api.faq-system.example.com/v1/faqs?page=2>; rel="next",
      <https://api.faq-system.example.com/v1/faqs?page=5>; rel="last"
```

#### レスポンス例
```json
{
  "total": 100,
  "page": 1,
  "per_page": 20,
  "sort": "created_at",
  "order": "desc",
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
      "updated_at": "2025-03-23T12:00:00Z",
      "status": 1,
      "_links": {
        "self": "/faqs/1",
        "edit": "/faqs/1/edit",
        "feedback": "/faqs/1/feedback"
      }
    }
  ]
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "invalid_parameter",
    "message": "不正なパラメータが指定されました",
    "details": {
      "page": "1以上の値を指定してください",
      "per_page": "100以下の値を指定してください"
    }
  }
}
```

### FAQ詳細取得
```http
GET /faqs/{id}
```

#### 必要な権限
- 公開FAQの取得: 認証不要
- 非公開FAQの取得: `read:faqs`権限
- 下書きFAQの取得: `manage:faqs`権限

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  }
}
```

#### キャッシュ戦略
- 公開FAQのみキャッシュ（1時間）
- 非公開・下書きFAQはキャッシュなし
- コンテンツ更新時にキャッシュ自動無効化

#### レスポンスヘッダー
```
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: "Sun, 23 Mar 2025 12:00:00 GMT"
```

#### レスポンス例
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

#### 必要な権限
- `create:faqs`権限（content_managerまたはadmin）

#### リクエストバリデーション
```json
{
  "title": {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "required": true,
    "description": "FAQタイトル"
  },
  "content": {
    "type": "string",
    "minLength": 1,
    "maxLength": 10000,
    "required": true,
    "description": "FAQ本文"
  },
  "category_ids": {
    "type": "array",
    "items": {
      "type": "integer",
      "minimum": 1
    },
    "minItems": 1,
    "maxItems": 5,
    "required": true,
    "description": "カテゴリID配列（1-5個）"
  },
  "tag_ids": {
    "type": "array",
    "items": {
      "type": "integer",
      "minimum": 1
    },
    "maxItems": 10,
    "description": "タグID配列（最大10個）"
  },
  "status": {
    "type": "integer",
    "enum": [0, 1, 2],
    "default": 0,
    "description": "ステータス（0:下書き, 1:公開, 2:非公開）"
  },
  "attachments": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "filename": {
          "type": "string",
          "maxLength": 255
        },
        "content_base64": {
          "type": "string",
          "maxLength": 10485760  // 10MB
        }
      }
    },
    "maxItems": 5,
    "description": "添付ファイル（最大5個、各10MBまで）"
  }
}
```

#### リクエスト例
```json
{
  "title": "アカウントの登録方法について",
  "content": "以下の手順でアカウントを登録できます...",
  "category_ids": [1, 2],
  "tag_ids": [1, 2],
  "status": 0,
  "attachments": [
    {
      "filename": "registration.png",
      "content_base64": "base64エンコードされたファイル内容"
    }
  ]
}
```

#### レート制限
- 1ユーザーあたり: 60リクエスト/時
- 添付ファイルの合計サイズ: 50MB/時

#### レスポンス例
```json
{
  "id": 1,
  "message": "FAQ created successfully",
  "faq": {
    "id": 1,
    "title": "アカウントの登録方法について",
    "status": 0,
    "created_at": "2025-03-23T12:00:00Z",
    "_links": {
      "self": "/faqs/1",
      "edit": "/faqs/1/edit"
    }
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "validation_error",
    "message": "入力内容に誤りがあります",
    "details": {
      "title": "タイトルは必須です",
      "category_ids": "少なくとも1つのカテゴリを選択してください",
      "attachments": "ファイルサイズは10MB以下にしてください"
    }
  }
}
```

### FAQ更新
```http
PUT /faqs/{id}
```

#### 必要な権限
- `update:faqs`権限（content_managerまたはadmin）
- 下書きFAQの更新: 作成者または`manage:faqs`権限
- 公開FAQの更新: `manage:faqs`権限

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  }
}
```

#### リクエストバリデーション
```json
{
  "title": {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "required": true,
    "description": "FAQタイトル"
  },
  "content": {
    "type": "string",
    "minLength": 1,
    "maxLength": 10000,
    "required": true,
    "description": "FAQ本文"
  },
  "category_ids": {
    "type": "array",
    "items": {
      "type": "integer",
      "minimum": 1
    },
    "minItems": 1,
    "maxItems": 5,
    "required": true,
    "description": "カテゴリID配列（1-5個）"
  },
  "tag_ids": {
    "type": "array",
    "items": {
      "type": "integer",
      "minimum": 1
    },
    "maxItems": 10,
    "description": "タグID配列（最大10個）"
  },
  "status": {
    "type": "integer",
    "enum": [0, 1, 2],
    "required": true,
    "description": "ステータス（0:下書き, 1:公開, 2:非公開）"
  },
  "attachments": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "description": "既存の添付ファイルID"
        },
        "filename": {
          "type": "string",
          "maxLength": 255
        },
        "content_base64": {
          "type": "string",
          "maxLength": 10485760  // 10MB
        }
      }
    },
    "maxItems": 5,
    "description": "添付ファイル（最大5個、各10MBまで）"
  }
}
```

#### リクエスト例
```json
{
  "title": "更新後のFAQタイトル",
  "content": "更新後のFAQ本文...",
  "category_ids": [1, 2],
  "tag_ids": [1, 2],
  "status": 1,
  "attachments": [
    {
      "id": 1  // 既存の添付ファイルを保持
    },
    {
      "filename": "new_image.png",
      "content_base64": "base64エンコードされたファイル内容"
    }
  ]
}
```

#### レート制限
- 1ユーザーあたり: 60リクエスト/時
- 添付ファイルの合計サイズ: 50MB/時

#### レスポンス例
```json
{
  "message": "FAQ updated successfully",
  "faq": {
    "id": 1,
    "title": "更新後のFAQタイトル",
    "status": 1,
    "updated_at": "2025-03-23T12:00:00Z",
    "_links": {
      "self": "/faqs/1",
      "edit": "/faqs/1/edit"
    }
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "validation_error",
    "message": "入力内容に誤りがあります",
    "details": {
      "title": "タイトルは必須です",
      "category_ids": "少なくとも1つのカテゴリを選択してください",
      "attachments": "添付ファイルが見つかりません（ID: 1）"
    }
  }
}
```

### FAQ一括操作
```http
POST /faqs/bulk
```

#### 必要な権限
- `manage:faqs`権限（adminのみ）

#### リクエストバリデーション
```json
{
  "action": {
    "type": "string",
    "enum": ["update_status", "delete", "categorize"],
    "required": true,
    "description": "実行する操作"
  },
  "faq_ids": {
    "type": "array",
    "items": {
      "type": "integer",
      "minimum": 1
    },
    "minItems": 1,
    "maxItems": 100,
    "required": true,
    "description": "対象のFAQ ID配列"
  },
  "params": {
    "type": "object",
    "properties": {
      "status": {
        "type": "integer",
        "enum": [0, 1, 2],
        "description": "update_status時の新しいステータス"
      },
      "category_ids": {
        "type": "array",
        "items": {
          "type": "integer",
          "minimum": 1
        },
        "description": "categorize時の新しいカテゴリID配列"
      }
    }
  }
}
```

#### リクエスト例
```json
{
  "action": "update_status",
  "faq_ids": [1, 2, 3, 4, 5],
  "params": {
    "status": 1
  }
}
```

#### レート制限
- 1ユーザーあたり: 10リクエスト/時

#### レスポンス例
```json
{
  "message": "Bulk operation completed",
  "results": {
    "total": 5,
    "succeeded": 4,
    "failed": 1,
    "failures": [
      {
        "faq_id": 3,
        "error": "FAQ not found"
      }
    ]
  }
}
```

### FAQバージョン履歴
```http
GET /faqs/{id}/versions
```

#### 必要な権限
- `read:faqs`権限（content_managerまたはadmin）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  }
}
```

#### レスポンス例
```json
{
  "versions": [
    {
      "version": 3,
      "editor": {
        "id": 1,
        "name": "管理者"
      },
      "changes": {
        "status": {
          "from": 0,
          "to": 1
        },
        "title": {
          "from": "下書きタイトル",
          "to": "公開タイトル"
        }
      },
      "created_at": "2025-03-23T12:00:00Z"
    },
    {
      "version": 2,
      "editor": {
        "id": 2,
        "name": "編集者"
      },
      "changes": {
        "content": {
          "modified": true
        }
      },
      "created_at": "2025-03-22T15:30:00Z"
    },
    {
      "version": 1,
      "editor": {
        "id": 2,
        "name": "編集者"
      },
      "changes": {
        "created": true
      },
      "created_at": "2025-03-22T10:00:00Z"
    }
  ]
}
```

### FAQバージョン復元
```http
POST /faqs/{id}/versions/{version}/restore
```

#### 必要な権限
- `manage:faqs`権限（adminのみ）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  },
  "version": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "復元するバージョン番号"
  }
}
```

#### レスポンス例
```json
{
  "message": "Version restored successfully",
  "faq": {
    "id": 1,
    "version": 4,
    "restored_from": 2,
    "updated_at": "2025-03-23T12:00:00Z"
  }
}
```

### FAQ削除
```http
DELETE /faqs/{id}
```

#### 必要な権限
- `delete:faqs`権限（content_managerまたはadmin）
- 下書きFAQの削除: 作成者または`manage:faqs`権限
- 公開/非公開FAQの削除: `manage:faqs`権限

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  }
}
```

#### 考慮事項
- 削除前に関連する以下のデータも削除
  - 添付ファイル
  - フィードバック
  - カテゴリ関連付け
  - タグ関連付け
- 履歴テーブルには削除記録を保持

#### レート制限
- 1ユーザーあたり: 30リクエスト/時

#### レスポンス例
```json
{
  "message": "FAQ deleted successfully",
  "faq": {
    "id": 1,
    "title": "削除されたFAQのタイトル",
    "deleted_at": "2025-03-23T12:00:00Z"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "delete_error",
    "message": "FAQを削除できません",
    "details": {
      "reason": "公開中のFAQは管理者権限が必要です"
    }
  }
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

#### 必要な権限
- 公開FAQへのフィードバック: 認証不要
- 非公開FAQへのフィードバック: `create:feedback`権限

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  }
}
```

#### リクエストバリデーション
```json
{
  "rating": {
    "type": "integer",
    "minimum": 1,
    "maximum": 5,
    "required": true,
    "description": "評価（1-5）"
  },
  "comment": {
    "type": "string",
    "maxLength": 1000,
    "description": "フィードバックコメント"
  },
  "is_resolved": {
    "type": "boolean",
    "description": "解決したかどうか"
  }
}
```

#### レート制限
- 未認証ユーザー: 10リクエスト/時/IP
- 認証済みユーザー: 30リクエスト/時

#### レスポンス例
```json
{
  "message": "フィードバックを受け付けました",
  "feedback": {
    "id": 1,
    "faq_id": 1,
    "rating": 5,
    "comment": "とても分かりやすい説明でした",
    "is_resolved": true,
    "created_at": "2025-03-23T12:00:00Z"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "invalid_feedback",
    "message": "不正なフィードバックです",
    "details": {
      "rating": "1から5の間の値を指定してください"
    }
  }
}
```

### フィードバック一覧取得
```http
GET /faqs/{id}/feedback
```

#### 必要な権限
- `read:feedback`権限（support_agent以上）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "FAQ ID"
  }
}
```

#### クエリパラメータバリデーション
```json
{
  "page": {
    "type": "integer",
    "minimum": 1,
    "default": 1,
    "description": "ページ番号"
  },
  "per_page": {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "default": 20,
    "description": "1ページの件数"
  },
  "sort": {
    "type": "string",
    "enum": ["created_at", "rating"],
    "default": "created_at",
    "description": "ソートフィールド"
  },
  "order": {
    "type": "string",
    "enum": ["asc", "desc"],
    "default": "desc",
    "description": "ソート順"
  }
}
```

#### レスポンス例
```json
{
  "total": 100,
  "page": 1,
  "per_page": 20,
  "feedback": [
    {
      "id": 1,
      "user": {
        "id": 1,
        "name": "ユーザー名"
      },
      "rating": 5,
      "comment": "とても分かりやすい説明でした",
      "is_resolved": true,
      "created_at": "2025-03-23T12:00:00Z"
    }
  ],
  "summary": {
    "average_rating": 4.5,
    "total_feedback": 100,
    "resolution_rate": 0.85
  }
}
```

## 7. 問合せ API ✉️

### エンドポイント概要
| エンドポイント | メソッド | 説明 |
|--------------|---------|------|
| /inquiries | POST | 新規問合せ作成 |
| /inquiries | GET | 問合せ一覧取得 |
| /inquiries/{id} | GET | 問合せ詳細取得 |
| /inquiries/{id}/responses | POST | 問合せへの返信 |
| /inquiries/{id}/status | PUT | 問合せステータス更新 |
| /inquiries/{id}/assign | PUT | 担当者割り当て |
| /inquiries/categories | GET | 問合せカテゴリ一覧取得 |

### 問合せ一覧取得
```http
GET /inquiries
```

#### 必要な権限
- 自身の問合せ一覧: `read:own_inquiries`権限
- 全問合せ一覧: `read:all_inquiries`権限（support_agent以上）

#### クエリパラメータバリデーション
```json
{
  "page": {
    "type": "integer",
    "minimum": 1,
    "default": 1,
    "description": "ページ番号"
  },
  "per_page": {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "default": 20,
    "description": "1ページの件数"
  },
  "status": {
    "type": "string",
    "enum": ["new", "in_progress", "waiting_response", "resolved", "closed"],
    "description": "ステータス"
  },
  "priority": {
    "type": "string",
    "enum": ["low", "medium", "high", "urgent"],
    "description": "優先度"
  },
  "category_id": {
    "type": "integer",
    "minimum": 1,
    "description": "問合せカテゴリID"
  },
  "assignee_id": {
    "type": "integer",
    "minimum": 1,
    "description": "担当者ID"
  },
  "search": {
    "type": "string",
    "maxLength": 100,
    "description": "件名・本文の全文検索"
  },
  "created_from": {
    "type": "string",
    "format": "date",
    "description": "作成日時の開始日（YYYY-MM-DD）"
  },
  "created_to": {
    "type": "string",
    "format": "date",
    "description": "作成日時の終了日（YYYY-MM-DD）"
  },
  "sort": {
    "type": "string",
    "enum": [
      "created_at",
      "updated_at",
      "priority",
      "status",
      "response_time"
    ],
    "default": "created_at",
    "description": "ソートフィールド"
  },
  "order": {
    "type": "string",
    "enum": ["asc", "desc"],
    "default": "desc",
    "description": "ソート順"
  }
}
```

#### レスポンスヘッダー
```
X-Total-Count: 100
X-Page-Number: 1
X-Page-Size: 20
Link: <https://api.faq-system.example.com/v1/inquiries?page=2>; rel="next",
      <https://api.faq-system.example.com/v1/inquiries?page=5>; rel="last"
```

#### レスポンス例
```json
{
  "total": 100,
  "page": 1,
  "per_page": 20,
  "inquiries": [
    {
      "id": 1,
      "ticket_number": "INQ20250323001",
      "subject": "アカウントログインについて",
      "status": "in_progress",
      "priority": "high",
      "category": {
        "id": 1,
        "name": "アカウント"
      },
      "user": {
        "id": 1,
        "name": "山田太郎",
        "email": "user@example.com"
      },
      "assignee": {
        "id": 2,
        "name": "サポート担当者A"
      },
      "response_count": 3,
      "last_response_at": "2025-03-23T12:30:00Z",
      "response_time": "30m",
      "created_at": "2025-03-23T12:00:00Z",
      "updated_at": "2025-03-23T12:30:00Z",
      "_links": {
        "self": "/inquiries/1",
        "responses": "/inquiries/1/responses"
      }
    }
  ],
  "summary": {
    "total_open": 80,
    "total_closed": 20,
    "by_status": {
      "new": 30,
      "in_progress": 40,
      "waiting_response": 10,
      "resolved": 15,
      "closed": 5
    },
    "by_priority": {
      "urgent": 5,
      "high": 15,
      "medium": 50,
      "low": 30
    },
    "by_category": [
      {
        "id": 1,
        "name": "アカウント",
        "count": 40
      }
    ],
    "average_response_time": "2h 30m",
    "average_resolution_time": "2d 4h"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "invalid_parameter",
    "message": "不正なパラメータが指定されました",
    "details": {
      "page": "1以上の値を指定してください",
      "created_from": "不正な日付形式です"
    }
  }
}
```
| /inquiries/{id} | GET | 問合せ詳細取得 |
| /inquiries/{id}/responses | POST | 問合せへの返信 |
| /inquiries/{id}/status | PUT | 問合せステータス更新 |
| /inquiries/{id}/assign | PUT | 担当者割り当て |
| /inquiries/categories | GET | 問合せカテゴリ一覧取得 |

### 問合せステータス一覧
| ステータス | 説明 |
|----------|------|
| new | 新規問合せ |
| in_progress | 対応中 |
| waiting_response | 返信待ち |
| resolved | 解決済み |
| closed | 完了 |

### 問合せ作成
```http
POST /inquiries
```

#### 必要な権限
- 認証済みユーザー: `create:inquiry`権限
- 未認証ユーザーからの問合せも許可（ただしレート制限あり）

#### リクエストバリデーション
```json
{
  "subject": {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "required": true,
    "description": "問合せ件名"
  },
  "content": {
    "type": "string",
    "minLength": 10,
    "maxLength": 10000,
    "required": true,
    "description": "問合せ内容"
  },
  "priority": {
    "type": "string",
    "enum": ["low", "medium", "high", "urgent"],
    "default": "medium",
    "description": "優先度"
  },
  "category_id": {
    "type": "integer",
    "minimum": 1,
    "description": "問合せカテゴリID"
  },
  "related_faq_id": {
    "type": "integer",
    "minimum": 1,
    "description": "関連するFAQ ID"
  },
  "contact_info": {
    "type": "object",
    "required": true,
    "properties": {
      "email": {
        "type": "string",
        "format": "email",
        "required": true,
        "description": "連絡先メールアドレス"
      },
      "name": {
        "type": "string",
        "maxLength": 100,
        "required": true,
        "description": "お名前"
      },
      "phone": {
        "type": "string",
        "pattern": "^[0-9-+]+$",
        "maxLength": 20,
        "description": "電話番号"
      }
    }
  },
  "attachments": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["filename", "content_base64"],
      "properties": {
        "filename": {
          "type": "string",
          "maxLength": 255,
          "pattern": "^[\\w\\-. ]+$"
        },
        "content_base64": {
          "type": "string",
          "maxLength": 5242880  // 5MB
        }
      }
    },
    "maxItems": 3,
    "description": "添付ファイル（最大3個、各5MBまで）"
  }
}
```

#### リクエスト例
```json
{
  "subject": "アカウントログインについて",
  "content": "アカウントにログインできません。パスワードをリセットしようとしても...",
  "priority": "high",
  "category_id": 1,
  "contact_info": {
    "email": "user@example.com",
    "name": "山田太郎",
    "phone": "090-1234-5678"
  },
  "attachments": [
    {
      "filename": "error_screenshot.png",
      "content_base64": "..."
    }
  ]
}
```

#### レート制限
- 未認証ユーザー: 3リクエスト/時/IP
- 認証済みユーザー: 10リクエスト/時
- 添付ファイルの合計サイズ: 15MB/時

#### レスポンス例
```json
{
  "message": "問合せを受け付けました",
  "inquiry": {
    "id": 1,
    "ticket_number": "INQ20250323001",
    "subject": "アカウントログインについて",
    "priority": "high",
    "status": "new",
    "category": {
      "id": 1,
      "name": "アカウント"
    },
    "estimated_response_time": "2025-03-23T14:00:00Z",
    "created_at": "2025-03-23T12:00:00Z",
    "_links": {
      "self": "/inquiries/1",
      "status": "/inquiries/1/status"
    }
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "validation_error",
    "message": "入力内容に誤りがあります",
    "details": {
      "subject": "件名は必須です",
      "content": "内容は10文字以上で入力してください",
      "attachments": [
        {
          "index": 0,
          "error": "ファイルサイズは5MB以下にしてください"
        }
      ]
    }
  }
}
```

### 問合せ詳細取得
```http
GET /inquiries/{id}
```

#### 必要な権限
- 自身の問合せ: `read:own_inquiries`権限
- 他者の問合せ: `read:all_inquiries`権限（support_agent以上）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "問合せID"
  }
}
```

#### レスポンス例
```json
{
  "id": 1,
  "ticket_number": "INQ20250323001",
  "subject": "アカウントログインについて",
  "content": "アカウントにログインできません...",
  "status": "in_progress",
  "priority": "high",
  "category": {
    "id": 1,
    "name": "アカウント"
  },
  "user": {
    "id": 1,
    "name": "山田太郎",
    "email": "user@example.com"
  },
  "assignee": {
    "id": 2,
    "name": "サポート担当者A",
    "email": "support@example.com"
  },
  "attachments": [
    {
      "id": 1,
      "filename": "error_screenshot.png",
      "mime_type": "image/png",
      "size": 1024,
      "url": "/attachments/1"
    }
  ],
  "responses": [
    {
      "id": 1,
      "content": "ご質問ありがとうございます...",
      "is_internal": false,
      "attachments": [],
      "author": {
        "id": 2,
        "name": "サポート担当者A",
        "is_staff": true
      },
      "created_at": "2025-03-23T12:30:00Z"
    }
  ],
  "timeline": [
    {
      "type": "created",
      "occurred_at": "2025-03-23T12:00:00Z",
      "actor": {
        "id": 1,
        "name": "山田太郎"
      }
    },
    {
      "type": "assigned",
      "occurred_at": "2025-03-23T12:15:00Z",
      "actor": {
        "id": 3,
        "name": "管理者"
      },
      "assignee": {
        "id": 2,
        "name": "サポート担当者A"
      }
    },
    {
      "type": "status_changed",
      "occurred_at": "2025-03-23T12:15:00Z",
      "actor": {
        "id": 2,
        "name": "サポート担当者A"
      },
      "from": "new",
      "to": "in_progress"
    }
  ],
  "created_at": "2025-03-23T12:00:00Z",
  "updated_at": "2025-03-23T12:30:00Z",
  "last_response_at": "2025-03-23T12:30:00Z",
  "_links": {
    "self": "/inquiries/1",
    "responses": "/inquiries/1/responses",
    "status": "/inquiries/1/status",
    "assign": "/inquiries/1/assign"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "not_found",
    "message": "指定された問合せが見つかりません",
    "details": {
      "id": "1"
    }
  }
}
```

### 問合せ返信作成
```http
POST /inquiries/{id}/responses
```

#### 必要な権限
- 自身の問合せへの返信: `create:own_inquiry_responses`権限
- 他者の問合せへの返信: `create:all_inquiry_responses`権限（support_agent以上）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "問合せID"
  }
}
```

#### リクエストバリデーション
```json
{
  "content": {
    "type": "string",
    "minLength": 1,
    "maxLength": 10000,
    "required": true,
    "description": "返信内容"
  },
  "is_internal_note": {
    "type": "boolean",
    "default": false,
    "description": "内部メモとして扱うかどうか（support_agent以上のみ設定可能）"
  },
  "status_update": {
    "type": "object",
    "description": "問合せステータスの更新（オプション）",
    "properties": {
      "status": {
        "type": "string",
        "enum": ["new", "in_progress", "waiting_response", "resolved", "closed"],
        "description": "新しいステータス"
      },
      "comment": {
        "type": "string",
        "maxLength": 1000,
        "description": "ステータス変更理由"
      }
    }
  },
  "attachments": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["filename", "content_base64"],
      "properties": {
        "filename": {
          "type": "string",
          "maxLength": 255,
          "pattern": "^[\\w\\-. ]+$"
        },
        "content_base64": {
          "type": "string",
          "maxLength": 5242880  // 5MB
        }
      }
    },
    "maxItems": 3,
    "description": "添付ファイル（最大3個、各5MBまで）"
  }
}
```

#### リクエスト例
```json
{
  "content": "ご迷惑をおかけし申し訳ございません。パスワードリセットの手順をご案内いたします...",
  "is_internal_note": false,
  "status_update": {
    "status": "in_progress",
    "comment": "対応を開始します"
  },
  "attachments": [
    {
      "filename": "password_reset_guide.pdf",
      "content_base64": "..."
    }
  ]
}
```

#### レート制限
- 認証済みユーザー: 30リクエスト/時
- 添付ファイルの合計サイズ: 15MB/時

#### レスポンス例
```json
{
  "message": "返信を送信しました",
  "response": {
    "id": 2,
    "content": "ご迷惑をおかけし申し訳ございません...",
    "is_internal_note": false,
    "attachments": [
      {
        "id": 2,
        "filename": "password_reset_guide.pdf",
        "mime_type": "application/pdf",
        "size": 2048,
        "url": "/attachments/2"
      }
    ],
    "author": {
      "id": 2,
      "name": "サポート担当者A",
      "is_staff": true
    },
    "created_at": "2025-03-23T13:00:00Z",
    "_links": {
      "inquiry": "/inquiries/1",
      "self": "/inquiries/1/responses/2"
    }
  },
  "status_update": {
    "from": "new",
    "to": "in_progress",
    "changed_at": "2025-03-23T13:00:00Z"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "validation_error",
    "message": "入力内容に誤りがあります",
    "details": {
      "content": "返信内容は必須です",
      "attachments": [
        {
          "index": 0,
          "error": "ファイルサイズは5MB以下にしてください"
        }
      ]
    }
  }
}
```
  "faq_id": {
    "type": "integer",
    "minimum": 1,
    "description": "関連するFAQ ID"
  },
  "category_id": {
    "type": "integer",
    "minimum": 1,
    "description": "問合せカテゴリID"
  },
  "contact_info": {
    "type": "object",
    "properties": {
      "email": {
        "type": "string",
        "format": "email",
        "required": true,
        "description": "連絡先メールアドレス"
      },
      "name": {
        "type": "string",
        "maxLength": 100,
        "required": true,
        "description": "お名前"
      },
      "phone": {
        "type": "string",
        "pattern": "^[0-9-+]+$",
        "maxLength": 20,
        "description": "電話番号"
      }
    }
  },
  "attachments": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "filename": {
          "type": "string",
          "maxLength": 255,
          "pattern": "^[\\w\\-. ]+$"
        },
        "content_base64": {
          "type": "string",
          "maxLength": 5242880  // 5MB
        }
      }
    },
    "maxItems": 3,
    "description": "添付ファイル（最大3個、各5MBまで）"
  }
}
```

#### レート制限
- 未認証ユーザー: 3リクエスト/時/IP
- 認証済みユーザー: 10リクエスト/時
- 添付ファイルの合計サイズ: 15MB/時

#### レスポンス例
```json
{
  "message": "問合せを受け付けました",
  "inquiry": {
    "id": 1,
    "ticket_number": "INQ20250323001",
    "subject": "アカウントについての質問",
    "priority": "medium",
    "status": "new",
    "created_at": "2025-03-23T12:00:00Z",
    "estimated_response_time": "2025-03-24T12:00:00Z",
    "_links": {
      "self": "/inquiries/1",
      "status": "/inquiries/1/status"
    }
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "validation_error",
    "message": "入力内容に誤りがあります",
    "details": {
      "subject": "件名は必須です",
      "content": "内容は10文字以上で入力してください",
      "attachments": "ファイルサイズは5MB以下にしてください"
    }
  }
}
```

### 問合せステータス更新
```http
PUT /inquiries/{id}/status
```

#### 必要な権限
- `update:inquiry_status`権限（support_agent以上）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "問合せID"
  }
}
```

#### リクエストバリデーション
```json
{
  "status": {
    "type": "string",
    "enum": ["new", "in_progress", "waiting_response", "resolved", "closed"],
    "required": true,
    "description": "新しいステータス"
  },
  "comment": {
    "type": "string",
    "maxLength": 1000,
    "description": "ステータス変更理由"
  },
  "notify_user": {
    "type": "boolean",
    "default": true,
    "description": "ユーザーに通知するかどうか"
  }
}
```

#### リクエスト例
```json
{
  "status": "resolved",
  "comment": "問題が解決しました",
  "notify_user": true
}
```

#### レート制限
- support_agent: 60リクエスト/時
- admin: 200リクエスト/時

#### レスポンス例
```json
{
  "message": "ステータスを更新しました",
  "inquiry": {
    "id": 1,
    "ticket_number": "INQ20250323001",
    "status": "resolved",
    "status_history": [
      {
        "from": "in_progress",
        "to": "resolved",
        "changed_by": {
          "id": 2,
          "name": "サポート担当者A"
        },
        "comment": "問題が解決しました",
        "changed_at": "2025-03-23T14:00:00Z"
      },
      {
        "from": "new",
        "to": "in_progress",
        "changed_by": {
          "id": 2,
          "name": "サポート担当者A"
        },
        "comment": "対応を開始します",
        "changed_at": "2025-03-23T13:00:00Z"
      }
    ],
    "updated_at": "2025-03-23T14:00:00Z",
    "_links": {
      "self": "/inquiries/1",
      "responses": "/inquiries/1/responses"
    }
  },
  "notifications": {
    "user_notified": true,
    "notification_sent_at": "2025-03-23T14:00:00Z"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "invalid_status_transition",
    "message": "このステータスへの変更はできません",
    "details": {
      "current_status": "new",
      "requested_status": "closed",
      "allowed_transitions": ["in_progress"]
    }
  }
}
```

### 問合せ担当者割り当て
```http
PUT /inquiries/{id}/assign
```

#### 必要な権限
- `assign:inquiry`権限（support_agent以上）

#### パスパラメータバリデーション
```json
{
  "id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "問合せID"
  }
}
```

#### リクエストバリデーション
```json
{
  "assignee_id": {
    "type": "integer",
    "minimum": 1,
    "required": true,
    "description": "割り当てる担当者のID"
  },
  "comment": {
    "type": "string",
    "maxLength": 1000,
    "description": "割り当ての理由や備考"
  },
  "notify_assignee": {
    "type": "boolean",
    "default": true,
    "description": "担当者に通知するかどうか"
  },
  "auto_status_update": {
    "type": "boolean",
    "default": true,
    "description": "自動的にステータスを'in_progress'に更新するかどうか"
  }
}
```

#### レート制限
- support_agent: 30リクエスト/時
- admin: 100リクエスト/時

#### レスポンス例
```json
{
  "message": "担当者を割り当てました",
  "inquiry": {
    "id": 1,
    "ticket_number": "INQ20250323001",
    "status": "in_progress",
    "assignee": {
      "id": 2,
      "name": "サポート担当者A",
      "email": "support_a@example.com",
      "department": "カスタマーサポート",
      "role": "support_agent"
    },
    "assignment_history": [
      {
        "assigned_to": {
          "id": 2,
          "name": "サポート担当者A"
        },
        "assigned_by": {
          "id": 3,
          "name": "管理者"
        },
        "assigned_at": "2025-03-23T15:00:00Z",
        "comment": "言語とフレームワークの経験が豊富な担当者に割り当て"
      }
    ],
    "status_update": {
      "from": "new",
      "to": "in_progress",
      "updated_at": "2025-03-23T15:00:00Z"
    },
    "updated_at": "2025-03-23T15:00:00Z",
    "_links": {
      "self": "/inquiries/1",
      "responses": "/inquiries/1/responses"
    }
  },
  "notifications": {
    "assignee_notified": true,
    "notification_sent_at": "2025-03-23T15:00:00Z"
  }
}
```

#### エラーレスポンス例
```json
{
  "error": {
    "code": "invalid_assignment",
    "message": "担当者の割り当てに失敗しました",
    "details": {
      "reason": "指定された担当者は既に最大担当件数に達しています",
      "current_assignments": 15,
      "max_assignments": 15
    }
  }
}
```

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