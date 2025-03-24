# ディレクトリ構成方針

## 1. 📁 基本方針

### プロジェクトルート構成
```
faq-system/
├── src/              # ソースコード
├── public/           # 静的ファイル
├── tests/            # テストファイル
├── docs/             # ドキュメント
├── scripts/          # ビルドスクリプト
└── config/           # 設定ファイル
```

## 2. 📂 ソースディレクトリ構成

### src/ ディレクトリ
```
src/
├── assets/           # 静的アセット
├── components/       # UIコンポーネント
├── features/         # 機能モジュール
├── hooks/           # カスタムフック
├── layouts/         # レイアウトコンポーネント
├── pages/           # ページコンポーネント
├── services/        # APIサービス
├── store/           # Reduxストア
├── styles/          # グローバルスタイル
├── types/           # 型定義
└── utils/           # ユーティリティ関数
```

### components/ ディレクトリ（Atomic Design）
```
components/
├── atoms/          # 基本要素
├── molecules/      # 分子レベルのコンポーネント
├── organisms/      # 有機体レベルのコンポーネント
└── templates/      # テンプレートコンポーネント
```

### features/ ディレクトリ
```
features/
├── faq/           # FAQ機能関連
├── chat/          # チャット機能関連
├── search/        # 検索機能関連
└── user/          # ユーザー管理関連
```

## 3. 🏷️ 命名規則

### ファイル命名
- **コンポーネント**: `PascalCase.tsx`
  - 例: `Button.tsx`, `SearchBar.tsx`

- **カスタムフック**: `use{Name}.ts`
  - 例: `useDebounce.ts`, `useAuth.ts`

- **ユーティリティ**: `camelCase.ts`
  - 例: `formatDate.ts`, `validateInput.ts`

- **型定義**: `{Name}.types.ts`
  - 例: `User.types.ts`, `FAQ.types.ts`

- **テスト**: `{Name}.test.tsx`
  - 例: `Button.test.tsx`, `useAuth.test.ts`

### ディレクトリ命名
- 機能モジュール: `kebab-case`
  - 例: `user-management/`, `faq-editor/`

- コンポーネントカテゴリ: `camelCase`
  - 例: `atoms/`, `molecules/`

## 4. 📋 モジュール構成

### コンポーネントディレクトリ構造
```
ComponentName/
├── index.ts            # エクスポート
├── ComponentName.tsx   # メインコンポーネント
├── ComponentName.styles.ts  # スタイル
├── ComponentName.test.tsx   # テスト
└── ComponentName.stories.tsx  # Storybook
```

### 機能モジュール構造
```
feature-name/
├── components/     # 機能固有のコンポーネント
├── hooks/         # 機能固有のフック
├── services/      # API通信
├── store/         # 状態管理
└── types/         # 型定義
```

## 5. 🔍 インポート規則

### インポートの順序
1. 外部ライブラリ
2. 内部モジュール（相対パス）
3. 型定義
4. スタイル

```typescript
// 外部ライブラリ
import React from 'react';
import { useDispatch } from 'react-redux';

// 内部モジュール
import { Button } from '@/components/atoms';
import { useAuth } from '@/hooks';

// 型定義
import { User } from '@/types';

// スタイル
import { StyledContainer } from './styles';
```

## 6. 🎨 スタイル管理

### グローバルスタイル構成
```
styles/
├── theme/         # テーマ設定
├── globals/       # グローバルスタイル
└── mixins/        # 共通スタイルミックスイン
```

### コンポーネントスタイル
- CSS-in-JSによる管理
- テーマトークンの使用
- レスポンシブミックスインの活用

## 7. 📦 状態管理構成

### Reduxストア構成
```
store/
├── index.ts       # ストア設定
├── rootReducer.ts # ルートリデューサー
└── slices/        # 機能別スライス
    ├── faq/
    ├── chat/
    ├── search/
    └── user/
```

## 8. 🌐 API通信構成

### サービスディレクトリ構成
```
services/
├── api/          # APIクライアント設定
├── endpoints/    # エンドポイント定義
└── types/        # API型定義
```

## 9. 🧪 テスト構成

### テストディレクトリ構成
```
tests/
├── unit/         # ユニットテスト
├── integration/  # 統合テスト
├── e2e/         # E2Eテスト
└── mocks/       # モックデータ
```

## 10. 📚 ドキュメント構成

### ドキュメントディレクトリ構成
```
docs/
├── api/          # API仕様
├── components/   # コンポーネント仕様
├── guides/       # 開発ガイド
└── architecture/ # アーキテクチャ文書