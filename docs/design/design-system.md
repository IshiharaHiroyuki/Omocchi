# FAQシステム デザインシステム

## 1. カラーシステム 🎨

### 1.1 基本カラーパレット

```css
:root {
  /* プライマリカラー */
  --primary-50: #E3F2FD;
  --primary-100: #BBDEFB;
  --primary-200: #90CAF9;
  --primary-300: #64B5F6;
  --primary-400: #42A5F5;
  --primary-500: #2196F3;  /* メインカラー */
  --primary-600: #1E88E5;
  --primary-700: #1976D2;
  --primary-800: #1565C0;
  --primary-900: #0D47A1;

  /* セカンダリカラー */
  --secondary-50: #F3E5F5;
  --secondary-100: #E1BEE7;
  --secondary-200: #CE93D8;
  --secondary-300: #BA68C8;
  --secondary-400: #AB47BC;
  --secondary-500: #9C27B0;  /* メインカラー */
  --secondary-600: #8E24AA;
  --secondary-700: #7B1FA2;
  --secondary-800: #6A1B9A;
  --secondary-900: #4A148C;

  /* グレースケール */
  --gray-50: #FAFAFA;
  --gray-100: #F5F5F5;
  --gray-200: #EEEEEE;
  --gray-300: #E0E0E0;
  --gray-400: #BDBDBD;
  --gray-500: #9E9E9E;
  --gray-600: #757575;
  --gray-700: #616161;
  --gray-800: #424242;
  --gray-900: #212121;

  /* セマンティックカラー */
  --success-main: #4CAF50;
  --success-light: #81C784;
  --success-dark: #388E3C;

  --warning-main: #FFC107;
  --warning-light: #FFD54F;
  --warning-dark: #FFA000;

  --error-main: #F44336;
  --error-light: #E57373;
  --error-dark: #D32F2F;

  --info-main: #2196F3;
  --info-light: #64B5F6;
  --info-dark: #1976D2;
}
```

### 1.2 状態カラー

```css
/* インタラクティブ状態 */
:root {
  --state-hover: rgba(0, 0, 0, 0.04);
  --state-selected: rgba(33, 150, 243, 0.08);
  --state-disabled: rgba(0, 0, 0, 0.38);
  --state-focus: rgba(33, 150, 243, 0.12);
}
```

## 2. タイポグラフィ 📝

### 2.1 フォントファミリー

```css
:root {
  /* システムフォント */
  --font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', 
                      'Hiragino Kaku Gothic ProN', 'Hiragino Sans', 
                      Meiryo, sans-serif;
  
  /* 見出し用フォント */
  --font-family-heading: 'Noto Sans JP', sans-serif;
  
  /* モノスペースフォント */
  --font-family-mono: 'Source Code Pro', monospace;
}
```

### 2.2 フォントサイズ

```css
:root {
  /* ベースサイズ: 16px */
  --font-size-xs: 0.75rem;   /* 12px */
  --font-size-sm: 0.875rem;  /* 14px */
  --font-size-base: 1rem;    /* 16px */
  --font-size-lg: 1.125rem;  /* 18px */
  --font-size-xl: 1.25rem;   /* 20px */
  --font-size-2xl: 1.5rem;   /* 24px */
  --font-size-3xl: 1.875rem; /* 30px */
  --font-size-4xl: 2.25rem;  /* 36px */
}
```

### 2.3 タイポグラフィスケール

```css
/* 見出し */
.h1 {
  font-family: var(--font-family-heading);
  font-size: var(--font-size-4xl);
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.02em;
}

.h2 {
  font-family: var(--font-family-heading);
  font-size: var(--font-size-3xl);
  font-weight: 700;
  line-height: 1.3;
  letter-spacing: -0.01em;
}

/* 本文 */
.body1 {
  font-family: var(--font-family-base);
  font-size: var(--font-size-base);
  line-height: 1.5;
  letter-spacing: 0.00938em;
}

.body2 {
  font-family: var(--font-family-base);
  font-size: var(--font-size-sm);
  line-height: 1.43;
  letter-spacing: 0.01071em;
}
```

## 3. スペーシングシステム 📏

### 3.1 基本スペーシング

```css
:root {
  /* ベースサイズ: 8px */
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-5: 1.5rem;   /* 24px */
  --space-6: 2rem;     /* 32px */
  --space-7: 2.5rem;   /* 40px */
  --space-8: 3rem;     /* 48px */
}
```

### 3.2 レイアウトグリッド

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: var(--space-4);
  padding: var(--space-4);
}

/* レスポンシブグリッド */
@media (max-width: 768px) {
  .grid-container {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

## 4. コンポーネントライブラリ 🧩

### 4.1 ボタン

```css
.button {
  /* ベーススタイル */
  padding: var(--space-2) var(--space-4);
  border-radius: 4px;
  font-weight: 500;
  transition: all 0.2s ease-in-out;

  /* バリエーション */
  &--primary {
    background-color: var(--primary-500);
    color: white;
    
    &:hover {
      background-color: var(--primary-600);
    }
  }

  &--secondary {
    background-color: var(--secondary-500);
    color: white;
    
    &:hover {
      background-color: var(--secondary-600);
    }
  }

  &--outlined {
    border: 1px solid var(--primary-500);
    color: var(--primary-500);
    
    &:hover {
      background-color: var(--state-hover);
    }
  }
}
```

### 4.2 フォーム要素

```css
.input {
  /* ベーススタイル */
  padding: var(--space-2);
  border: 1px solid var(--gray-300);
  border-radius: 4px;
  font-size: var(--font-size-base);
  transition: all 0.2s ease-in-out;

  &:focus {
    border-color: var(--primary-500);
    box-shadow: 0 0 0 2px var(--state-focus);
  }

  &:disabled {
    background-color: var(--gray-100);
    color: var(--state-disabled);
  }
}
```

## 5. シャドウシステム 🌓

```css
:root {
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}
```

## 6. アニメーション 🎬

### 6.1 トランジション

```css
:root {
  --transition-fast: 150ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-normal: 200ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-slow: 300ms cubic-bezier(0.4, 0, 0.2, 1);
}
```

### 6.2 アニメーションプリセット

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
```

## 7. アイコンシステム 🎯

```css
.icon {
  /* ベースサイズ */
  --icon-size-sm: 16px;
  --icon-size-md: 24px;
  --icon-size-lg: 32px;

  /* アイコンカラー */
  --icon-color: currentColor;
  
  width: var(--icon-size-md);
  height: var(--icon-size-md);
  fill: var(--icon-color);
}
```

## 8. Z-indexシステム 📚

```css
:root {
  --z-negative: -1;
  --z-normal: 0;
  --z-dropdown: 1000;
  --z-sticky: 1200;
  --z-fixed: 1300;
  --z-modal: 1400;
  --z-popover: 1500;
  --z-tooltip: 1600;
}
```

## 9. ブレークポイント 📱

```css
:root {
  --breakpoint-sm: 640px;   /* スマートフォン */
  --breakpoint-md: 768px;   /* タブレット */
  --breakpoint-lg: 1024px;  /* ノートPC */
  --breakpoint-xl: 1280px;  /* デスクトップ */
  --breakpoint-2xl: 1536px; /* ワイドスクリーン */
}

/* メディアクエリミックスイン */
@mixin sm {
  @media (min-width: var(--breakpoint-sm)) { @content; }
}

@mixin md {
  @media (min-width: var(--breakpoint-md)) { @content; }
}

@mixin lg {
  @media (min-width: var(--breakpoint-lg)) { @content; }
}
```

## 10. アクセシビリティ ♿

### 10.1 フォーカス状態

```css
:root {
  --focus-ring-color: var(--primary-500);
  --focus-ring-width: 2px;
  --focus-ring-offset: 2px;
}

/* フォーカスリング */
.focus-ring {
  &:focus-visible {
    outline: var(--focus-ring-width) solid var(--focus-ring-color);
    outline-offset: var(--focus-ring-offset);
  }
}
```

### 10.2 Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## 11. ダークモード 🌙

```css
:root[data-theme="dark"] {
  /* ダークモード用カラー */
  --background: #121212;
  --surface: #1E1E1E;
  --text-primary: rgba(255, 255, 255, 0.87);
  --text-secondary: rgba(255, 255, 255, 0.60);
  
  /* ダークモード用シャドウ */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.8);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.8);
  
  /* ダークモード用ステート */
  --state-hover: rgba(255, 255, 255, 0.08);
  --state-selected: rgba(33, 150, 243, 0.16);
}
```

## 12. 使用例 💡

### 12.1 カード要素

```css
.card {
  background-color: white;
  border-radius: 8px;
  padding: var(--space-4);
  box-shadow: var(--shadow-md);
  transition: var(--transition-normal);

  &:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-lg);
  }

  &__title {
    font-size: var(--font-size-xl);
    color: var(--gray-900);
    margin-bottom: var(--space-2);
  }

  &__content {
    font-size: var(--font-size-base);
    color: var(--gray-700);
  }
}
```

### 12.2 ナビゲーション

```css
.nav {
  display: flex;
  gap: var(--space-4);
  padding: var(--space-4);
  background-color: var(--primary-500);

  &__item {
    color: white;
    font-size: var(--font-size-base);
    padding: var(--space-2) var(--space-4);
    border-radius: 4px;
    transition: var(--transition-fast);

    &:hover {
      background-color: var(--primary-600);
    }

    &--active {
      background-color: var(--primary-700);
    }
  }
}