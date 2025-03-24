# フォームコンポーネント設計書

## 1. 基本フォーム要素 📝

### 1.1 テキスト入力

```css
/* 基本スタイル */
.form-input {
  /* レイアウト */
  width: 100%;
  padding: var(--space-3);
  border-radius: var(--radius-md);
  
  /* 見た目 */
  border: 1px solid var(--gray-300);
  background-color: white;
  
  /* タイポグラフィ */
  font-size: var(--font-size-base);
  line-height: 1.5;
  
  /* トランジション */
  transition: all 0.2s ease;
  
  /* 状態 */
  &:hover {
    border-color: var(--gray-400);
  }
  
  &:focus {
    border-color: var(--primary-500);
    box-shadow: 0 0 0 2px var(--primary-100);
    outline: none;
  }
  
  &:disabled {
    background-color: var(--gray-100);
    color: var(--gray-500);
    cursor: not-allowed;
  }
}

/* サイズバリエーション */
.form-input--sm {
  padding: var(--space-2);
  font-size: var(--font-size-sm);
}

.form-input--lg {
  padding: var(--space-4);
  font-size: var(--font-size-lg);
}
```

### 1.2 セレクトボックス

```css
.form-select {
  /* 基本スタイル継承 */
  @extend .form-input;
  
  /* セレクト固有スタイル */
  padding-right: var(--space-8);
  appearance: none;
  background-image: url("data:image/svg+xml,..."); /* カスタム矢印 */
  background-repeat: no-repeat;
  background-position: right var(--space-3) center;
  
  /* オプショングループ */
  optgroup {
    font-weight: 600;
    color: var(--gray-700);
  }
}
```

### 1.3 チェックボックス・ラジオボタン

```css
.form-checkbox,
.form-radio {
  /* アクセシビリティのための隠し要素 */
  &__input {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    border: 0;
  }
  
  /* カスタムコントロール */
  &__control {
    position: relative;
    display: inline-block;
    width: 1.25rem;
    height: 1.25rem;
    border: 2px solid var(--gray-400);
    background-color: white;
    transition: all 0.2s ease;
  }
  
  /* チェック状態 */
  &__input:checked + &__control {
    border-color: var(--primary-500);
    background-color: var(--primary-500);
    
    &::after {
      content: "";
      position: absolute;
      /* チェックマーク/ドットのスタイル */
    }
  }
}
```

## 2. フォームレイアウト 📐

### 2.1 フォームグループ

```css
.form-group {
  margin-bottom: var(--space-4);
  
  /* ラベル */
  &__label {
    display: block;
    margin-bottom: var(--space-2);
    font-weight: 500;
    color: var(--gray-700);
  }
  
  /* 必須マーク */
  &__required {
    color: var(--error-500);
    margin-left: var(--space-1);
  }
  
  /* ヘルプテキスト */
  &__help {
    margin-top: var(--space-1);
    font-size: var(--font-size-sm);
    color: var(--gray-600);
  }
}
```

### 2.2 インラインフォーム

```css
.form-inline {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  
  @media (max-width: 767px) {
    flex-direction: column;
    align-items: stretch;
  }
}
```

## 3. バリデーション表示 ⚠️

### 3.1 エラー状態

```css
.form-input--error {
  border-color: var(--error-500);
  
  &:hover,
  &:focus {
    border-color: var(--error-600);
    box-shadow: 0 0 0 2px var(--error-100);
  }
}

.form-error {
  display: flex;
  align-items: center;
  margin-top: var(--space-1);
  color: var(--error-500);
  font-size: var(--font-size-sm);
  
  &__icon {
    margin-right: var(--space-1);
  }
}
```

### 3.2 成功状態

```css
.form-input--success {
  border-color: var(--success-500);
  
  &:hover,
  &:focus {
    border-color: var(--success-600);
    box-shadow: 0 0 0 2px var(--success-100);
  }
}

.form-success {
  display: flex;
  align-items: center;
  margin-top: var(--space-1);
  color: var(--success-500);
  font-size: var(--font-size-sm);
}
```

## 4. インタラクティブフィードバック 🔄

### 4.1 バリデーションインジケータ

```css
.validation-indicator {
  position: absolute;
  right: var(--space-3);
  top: 50%;
  transform: translateY(-50%);
  transition: all 0.2s ease;
  
  /* ローディング状態 */
  &--loading {
    animation: spin 1s linear infinite;
  }
  
  /* 成功状態 */
  &--success {
    color: var(--success-500);
  }
  
  /* エラー状態 */
  &--error {
    color: var(--error-500);
  }
}
```

### 4.2 インラインバリデーション

```css
.inline-validation {
  position: relative;
  
  &__message {
    position: absolute;
    top: 100%;
    left: 0;
    margin-top: var(--space-1);
    padding: var(--space-2);
    border-radius: var(--radius-sm);
    background-color: white;
    box-shadow: var(--shadow-lg);
    z-index: 10;
    
    /* アニメーション */
    animation: slideIn 0.2s ease;
  }
}
```

## 5. 高度な入力要素 🎛️

### 5.1 タグ入力

```css
.tag-input {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
  padding: var(--space-2);
  border: 1px solid var(--gray-300);
  border-radius: var(--radius-md);
  
  &__tag {
    display: inline-flex;
    align-items: center;
    padding: var(--space-1) var(--space-2);
    background-color: var(--primary-100);
    border-radius: var(--radius-full);
    font-size: var(--font-size-sm);
    
    &__remove {
      margin-left: var(--space-1);
      cursor: pointer;
    }
  }
  
  &__input {
    border: none;
    outline: none;
    flex: 1;
    min-width: 120px;
  }
}
```

### 5.2 数値入力

```css
.number-input {
  display: flex;
  align-items: center;
  
  &__control {
    width: var(--space-8);
    text-align: center;
    -moz-appearance: textfield;
    
    &::-webkit-inner-spin-button,
    &::-webkit-outer-spin-button {
      -webkit-appearance: none;
    }
  }
  
  &__button {
    padding: var(--space-2);
    border: 1px solid var(--gray-300);
    background-color: var(--gray-50);
    cursor: pointer;
    
    &:hover {
      background-color: var(--gray-100);
    }
  }
}
```

## 6. アクセシビリティ対応 ♿

### 6.1 フォーカス管理

```css
/* フォーカスリング */
:focus-visible {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
}

/* スキップリンク */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: var(--space-2);
  background-color: var(--primary-500);
  color: white;
  
  &:focus {
    top: 0;
  }
}
```

### 6.2 エラー通知

```css
.error-announcement {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
  
  /* スクリーンリーダー用 */
  &[aria-live="polite"] {
    /* エラーメッセージ更新時に読み上げ */
  }
}
```

## 7. レスポンシブ対応 📱

```css
/* フォームレイアウト */
@media (max-width: 767px) {
  .form-group {
    margin-bottom: var(--space-3);
  }
  
  .form-inline {
    flex-direction: column;
  }
  
  /* タッチターゲットサイズ */
  .form-input,
  .form-select,
  .form-checkbox__control,
  .form-radio__control {
    min-height: 44px;
  }
}
```

## 8. 使用例 💡

### 8.1 問い合わせフォーム

```html
<form class="form">
  <div class="form-group">
    <label class="form-group__label">
      お名前
      <span class="form-group__required">*</span>
    </label>
    <input type="text" class="form-input" required />
    <p class="form-group__help">
      姓名の間にスペースを入れてください
    </p>
  </div>
  
  <div class="form-group">
    <label class="form-group__label">
      メールアドレス
      <span class="form-group__required">*</span>
    </label>
    <input type="email" class="form-input" required />
  </div>
  
  <div class="form-group">
    <label class="form-group__label">
      お問い合わせ内容
    </label>
    <textarea class="form-input" rows="5"></textarea>
  </div>
  
  <button type="submit" class="button button--primary">
    送信する
  </button>
</form>