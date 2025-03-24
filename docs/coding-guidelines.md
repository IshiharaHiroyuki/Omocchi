# コーディング規約 & 命名規則ガイドライン

## 📝 目次

1. [基本方針](#基本方針)
2. [命名規則](#命名規則)
3. [コードフォーマット](#コードフォーマット)
4. [TypeScript / JavaScript](#typescript--javascript)
5. [Redux](#redux)
6. [エラーハンドリング](#エラーハンドリング)
7. [Reactコンポーネント](#reactコンポーネント)
8. [テスト](#テスト)
9. [コメント・ドキュメンテーション](#コメントドキュメンテーション)
10. [セキュリティ](#セキュリティ)
11. [パフォーマンス](#パフォーマンス)
12. [レビューチェックリスト](#レビューチェックリスト)

## 1. 基本方針

### 1.1 設計原則
- **単一責任の原則**: 各モジュール・クラス・関数は1つの明確な責任のみを持つ
- **DRY原則**: コードの重複を避け、再利用可能なモジュールを作成
- **KISS原則**: シンプルで理解しやすい実装を心がける
- **YAGNI原則**: 現時点で必要のない機能は実装しない

### 1.2 コード品質
- 可読性を最優先
- 自己文書化コード
- テスト容易性の確保
- パフォーマンスへの配慮

## 2. 命名規則

### 2.1 一般規則
- 意図が明確に伝わる命名を使用
- 略語は一般的なもののみ使用
- 単語の省略は避ける

### 2.2 識別子の命名規則

#### ファイル名
```typescript
// コンポーネント（PascalCase）
Button.tsx
SearchBar.tsx

// カスタムフック（useから始める）
useAuth.ts
useDebounce.ts

// ユーティリティ（camelCase）
formatDate.ts
validateInput.ts

// 型定義
types.ts
User.types.ts

// テスト
Button.test.tsx
useAuth.test.ts
```

#### 変数名
```typescript
// 通常の変数（camelCase）
const userName = 'John';
const itemCount = 42;

// 真偽値（is, has, shouldなどから始める）
const isLoading = true;
const hasPermission = false;
const shouldUpdate = true;

// 定数（SNAKE_CASE）
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';
```

#### 関数名
```typescript
// 動詞から始める
function getUserData() { ... }
function validateInput() { ... }
function handleSubmit() { ... }

// イベントハンドラ（handleから始める）
const handleClick = () => { ... }
const handleChange = () => { ... }
```

#### インターフェース・型
```typescript
// インターフェース（Iプレフィックスは使用しない）
interface User {
  id: string;
  name: string;
  email: string;
}

// 型（具体的な名前をつける）
type ButtonVariant = 'primary' | 'secondary' | 'ghost';
type InputChangeHandler = (event: ChangeEvent<HTMLInputElement>) => void;
```

## 3. コードフォーマット

### 3.1 基本ルール
```typescript
// インデント: スペース2個
function example() {
  const value = 1;
  if (value === 1) {
    return true;
  }
}

// 1行の最大長: 80文字
// 長い行は適切に改行
const longString = 
  'This is a very long string that should be ' +
  'broken into multiple lines for readability';

// カンマ、コロンの後にスペース
const obj = { name: 'John', age: 30 };
```

### 3.2 括弧・スペース
```typescript
// 制御構文の括弧前後にスペース
if (condition) {
  // code
}

// 関数呼び出しの括弧前にスペースなし
function getName() {
  return 'John';
}
getName();

// オブジェクトリテラルの括弧内にスペース
const user = { name: 'John' };
```

## 4. TypeScript / JavaScript

### 4.1 型定義
```typescript
// 明示的な型定義
const name: string = 'John';
const age: number = 30;

// ジェネリック型の活用
function getFirst<T>(array: T[]): T | undefined {
  return array[0];
}

// Union型の活用
type Status = 'idle' | 'loading' | 'success' | 'error';
```

### 4.2 非同期処理
```typescript
// async/awaitの使用
async function fetchUser(id: string) {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    handleError(error);
    throw error;
  }
}

// Promise.allの活用
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts()
]);
```

## 5. Redux

### 5.1 Storeの構成
```typescript
// スライスの定義
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface UserState {
  data: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  data: null,
  isLoading: false,
  error: null,
};

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.data = action.payload;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.isLoading = action.payload;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
    },
  },
});
```

### 5.2 非同期アクション
```typescript
// createAsyncThunkの使用
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId: string, { rejectWithValue }) => {
    try {
      const response = await api.get(`/users/${userId}`);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState,
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.isLoading = true;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload as string;
      });
  },
});
```

### 5.3 セレクター
```typescript
// 再利用可能なセレクター
import { createSelector } from '@reduxjs/toolkit';

const selectUserState = (state: RootState) => state.user;

export const selectUser = createSelector(
  selectUserState,
  (userState) => userState.data
);

export const selectIsLoading = createSelector(
  selectUserState,
  (userState) => userState.isLoading
);

// 複雑なセレクターの例
export const selectUserPermissions = createSelector(
  selectUser,
  (user) => user?.permissions ?? []
);
```

## 6. エラーハンドリング

### 6.1 グローバルエラーハンドリング
```typescript
// エラーバウンダリー
class ErrorBoundary extends React.Component<
  { children: ReactNode },
  { hasError: boolean }
> {
  constructor(props: { children: ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // エラーログの送信
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

### 6.2 APIエラーハンドリング
```typescript
// カスタムエラークラス
class APIError extends Error {
  constructor(
    message: string,
    public status: number,
    public code: string
  ) {
    super(message);
    this.name = 'APIError';
  }
}

// エラーハンドリングフック
function useAPIError() {
  const handleError = useCallback((error: unknown) => {
    if (error instanceof APIError) {
      switch (error.status) {
        case 401:
          // 認証エラー処理
          navigateToLogin();
          break;
        case 403:
          // 権限エラー処理
          showPermissionError();
          break;
        case 404:
          // Not Found処理
          showNotFound();
          break;
        default:
          // その他のエラー処理
          showGeneralError(error.message);
      }
    } else {
      // 予期せぬエラーの処理
      showUnexpectedError();
    }
  }, []);

  return { handleError };
}
```

### 6.3 フォームバリデーション
```typescript
// バリデーションルール
const validationRules = {
  required: (value: unknown) =>
    value !== undefined && value !== '' || '必須項目です',
  email: (value: string) =>
    /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value) ||
    '有効なメールアドレスを入力してください',
  password: (value: string) =>
    /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$/.test(value) ||
    'パスワードは8文字以上で、文字と数字を含む必要があります',
};

// バリデーション実行
function validateForm<T extends Record<string, unknown>>(
  values: T,
  rules: Record<keyof T, Array<(value: unknown) => true | string>>
): Record<keyof T, string | undefined> {
  const errors = {} as Record<keyof T, string | undefined>;

  Object.keys(rules).forEach((key) => {
    const value = values[key];
    const fieldRules = rules[key];

    for (const rule of fieldRules) {
      const result = rule(value);
      if (result !== true) {
        errors[key] = result;
        break;
      }
    }
  });

  return errors;
}
```

## 7. Reactコンポーネント

### 5.1 コンポーネント定義
```typescript
// 関数コンポーネントを使用
const Button: FC<ButtonProps> = ({ 
  children, 
  variant = 'primary',
  onClick 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// Propsの型定義
interface ButtonProps {
  children: ReactNode;
  variant?: 'primary' | 'secondary';
  onClick?: () => void;
}
```

### 5.2 Hooks
```typescript
// カスタムHook
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  return [storedValue, setStoredValue] as const;
}
```

## 6. テスト

### 6.1 ユニットテスト
```typescript
describe('Button', () => {
  it('クリックイベントが発火する', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### 6.2 テストカバレッジ
- ユニットテストカバレッジ目標: 80%以上
- 重要なビジネスロジックは100%のカバレッジを目指す
- E2Eテストは重要なユーザーフローをカバー

## 7. コメント・ドキュメンテーション

### 7.1 コードコメント
```typescript
// 基本的なコメント
// 変数の目的や制約を説明
const MAX_RETRIES = 3; // API呼び出しの最大リトライ回数

// 関数のドキュメント
/**
 * ユーザーデータを取得する
 * @param id ユーザーID
 * @returns ユーザーオブジェクト
 * @throws {ApiError} API呼び出しが失敗した場合
 */
async function fetchUser(id: string): Promise<User> {
  // 実装
}
```

### 7.2 TODO コメント
```typescript
// TODO: 後で実装する機能や改善点を記述
// TODO: キャッシュの有効期限を設定する
// TODO: エラーハンドリングを改善する
```

## 8. セキュリティ

### 8.1 入力検証
```typescript
// ユーザー入力の検証
function validateInput(input: string): boolean {
  // XSS対策
  const sanitizedInput = DOMPurify.sanitize(input);
  // 入力値の検証ロジック
  return /^[a-zA-Z0-9\s]{1,100}$/.test(sanitizedInput);
}
```

### 8.2 認証・認可
```typescript
// 認証状態の確認
function requireAuth(role: UserRole) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    // 認証・認可ロジック
  };
}
```

## 9. パフォーマンス

### 9.1 最適化
```typescript
// メモ化の活用
const MemoizedComponent = React.memo(({ data }) => {
  return <div>{data}</div>;
});

// useMemoの適切な使用
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(deps);
}, [deps]);
```

### 9.2 バンドルサイズ
- コードスプリッティングの活用
- 必要な依存関係のみをインポート
- 画像・アセットの最適化

## 10. レビューチェックリスト

### 10.1 コードレビュー基準
- [ ] 命名規則に準拠しているか
- [ ] 適切な型定義がされているか
- [ ] エラーハンドリングが適切か
- [ ] テストが十分に書かれているか
- [ ] パフォーマンスへの考慮があるか
- [ ] セキュリティ上の問題がないか
- [ ] ドキュメントが更新されているか

### 10.2 プルリクエスト要件
- 適切な粒度（1つの機能・修正につき1つのPR）
- 明確な説明文
- テストの実行結果
- 関連するドキュメントの更新