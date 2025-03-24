# レスポンシブデザインシステム

## 1. ブレークポイントシステム 📱

### 1.1 デバイス別ブレークポイント

```css
:root {
  /* メインブレークポイント */
  --breakpoint-xs: 0;          /* 320px~ スマートフォン（小） */
  --breakpoint-sm: 640px;      /* スマートフォン（大） */
  --breakpoint-md: 768px;      /* タブレット（縦） */
  --breakpoint-lg: 1024px;     /* タブレット（横）/ ノートPC */
  --breakpoint-xl: 1280px;     /* デスクトップ */
  --breakpoint-2xl: 1536px;    /* ワイドスクリーン */

  /* 補助ブレークポイント */
  --breakpoint-mobile: 480px;  /* モバイル最大幅 */
  --breakpoint-tablet: 960px;  /* タブレット最適幅 */
  --breakpoint-desktop: 1440px; /* デスクトップ最適幅 */
}
```

### 1.2 メディアクエリミックスイン

```scss
// ブレークポイントミックスイン
@mixin responsive($breakpoint) {
  @if $breakpoint == xs {
    @media (min-width: 0) { @content; }
  }
  @if $breakpoint == sm {
    @media (min-width: 640px) { @content; }
  }
  @if $breakpoint == md {
    @media (min-width: 768px) { @content; }
  }
  @if $breakpoint == lg {
    @media (min-width: 1024px) { @content; }
  }
  @if $breakpoint == xl {
    @media (min-width: 1280px) { @content; }
  }
  @if $breakpoint == 2xl {
    @media (min-width: 1536px) { @content; }
  }
}

// 使用例
.example {
  @include responsive(sm) {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
  
  @include responsive(md) {
    grid-template-columns: repeat(3, 1fr);
  }
  
  @include responsive(lg) {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

## 2. グリッドシステム 🔲

### 2.1 基本グリッド構造

```css
.container {
  /* コンテナ幅設定 */
  --container-margin: 1rem;
  --container-max-width: min(100% - var(--container-margin) * 2, var(--breakpoint-desktop));
  
  width: var(--container-max-width);
  margin-inline: auto;
  padding-inline: var(--container-margin);
}

.grid {
  display: grid;
  gap: var(--grid-gap, 1rem);
  
  /* 基本12カラムグリッド */
  grid-template-columns: repeat(12, 1fr);
}
```

### 2.2 レスポンシブグリッドシステム

```css
/* グリッドレイアウトクラス */
.grid {
  /* モバイルファースト: 1カラム */
  grid-template-columns: 1fr;
  
  /* スマートフォン（大）: 2カラム */
  @media (min-width: 640px) {
    grid-template-columns: repeat(2, 1fr);
  }
  
  /* タブレット: 3カラム */
  @media (min-width: 768px) {
    grid-template-columns: repeat(3, 1fr);
  }
  
  /* デスクトップ: 4カラム */
  @media (min-width: 1024px) {
    grid-template-columns: repeat(4, 1fr);
  }
}

/* グリッドカラム数制御クラス */
.col-span-1 { grid-column: span 1; }
.col-span-2 { grid-column: span 2; }
.col-span-3 { grid-column: span 3; }
.col-span-4 { grid-column: span 4; }
.col-span-5 { grid-column: span 5; }
.col-span-6 { grid-column: span 6; }
.col-span-7 { grid-column: span 7; }
.col-span-8 { grid-column: span 8; }
.col-span-9 { grid-column: span 9; }
.col-span-10 { grid-column: span 10; }
.col-span-11 { grid-column: span 11; }
.col-span-12 { grid-column: span 12; }
```

### 2.3 コンテナ幅設定

```css
/* コンテナサイズ変数 */
:root {
  /* 固定幅コンテナ */
  --container-sm: 640px;   /* スマートフォン */
  --container-md: 768px;   /* タブレット */
  --container-lg: 1024px;  /* ノートPC */
  --container-xl: 1280px;  /* デスクトップ */
  
  /* 余白設定 */
  --container-padding-sm: 1rem;
  --container-padding-md: 2rem;
  --container-padding-lg: 3rem;
  
  /* 最大幅制限 */
  --content-max-width: 1920px;
}

/* コンテナクラス */
.container {
  width: 100%;
  margin-inline: auto;
  padding-inline: var(--container-padding-sm);
  
  @media (min-width: 640px) {
    max-width: var(--container-sm);
    padding-inline: var(--container-padding-md);
  }
  
  @media (min-width: 768px) {
    max-width: var(--container-md);
  }
  
  @media (min-width: 1024px) {
    max-width: var(--container-lg);
    padding-inline: var(--container-padding-lg);
  }
  
  @media (min-width: 1280px) {
    max-width: var(--container-xl);
  }
}
```

## 3. レイアウトパターン 📐

### 3.1 基本レイアウト

```css
/* 2カラムレイアウト */
.layout-2-column {
  display: grid;
  gap: var(--space-4);
  
  @media (min-width: 768px) {
    grid-template-columns: 2fr 1fr;
  }
}

/* 3カラムレイアウト */
.layout-3-column {
  display: grid;
  gap: var(--space-4);
  
  @media (min-width: 1024px) {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* サイドバーレイアウト */
.layout-sidebar {
  display: grid;
  gap: var(--space-4);
  
  @media (min-width: 768px) {
    grid-template-columns: 250px 1fr;
  }
  
  @media (min-width: 1024px) {
    grid-template-columns: 300px 1fr;
  }
}
```

### 3.2 アダプティブコンポーネント

```css
/* カード表示 */
.card-grid {
  display: grid;
  gap: var(--space-4);
  
  /* レスポンシブグリッド */
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 300px), 1fr));
}

/* フレックスレイアウト */
.flex-container {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-4);
  
  > * {
    flex: 1 1 var(--flex-basis, 300px);
    min-width: 0;
  }
}
```

## 4. ユーティリティクラス 🛠️

### 4.1 表示制御

```css
/* 表示/非表示 */
.hidden-xs { @media (max-width: 639px) { display: none !important; } }
.hidden-sm { @media (min-width: 640px) and (max-width: 767px) { display: none !important; } }
.hidden-md { @media (min-width: 768px) and (max-width: 1023px) { display: none !important; } }
.hidden-lg { @media (min-width: 1024px) and (max-width: 1279px) { display: none !important; } }
.hidden-xl { @media (min-width: 1280px) { display: none !important; } }

/* 表示順序 */
.order-first { order: -1; }
.order-last { order: 999; }
.order-none { order: 0; }
```

### 4.2 スペーシング調整

```css
/* マージン制御 */
.m-auto { margin: auto; }
.mx-auto { margin-inline: auto; }
.my-auto { margin-block: auto; }

/* パディング制御 */
.p-responsive {
  padding: var(--space-2);
  
  @media (min-width: 768px) {
    padding: var(--space-4);
  }
  
  @media (min-width: 1024px) {
    padding: var(--space-6);
  }
}
```

## 5. レスポンシブ画像 🖼️

```css
/* レスポンシブ画像基本設定 */
.img-responsive {
  max-width: 100%;
  height: auto;
}

/* アスペクト比維持 */
.img-container {
  position: relative;
  width: 100%;
  
  &::before {
    content: "";
    display: block;
    padding-top: var(--aspect-ratio, 56.25%); /* 16:9 デフォルト */
  }
  
  img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}
```

## 6. タイポグラフィ調整 📝

```css
/* レスポンシブフォントサイズ */
:root {
  /* ベースサイズ調整 */
  font-size: 14px;
  
  @media (min-width: 768px) {
    font-size: 15px;
  }
  
  @media (min-width: 1024px) {
    font-size: 16px;
  }
}

/* Fluid Typography */
.fluid-type {
  font-size: clamp(
    var(--fluid-min, 1rem),
    var(--fluid-val, 3vw),
    var(--fluid-max, 2rem)
  );
}
```

## 7. パフォーマンス考慮 🚀

```css
/* 画像最適化 */
.img-responsive {
  width: 100%;
  height: auto;
  
  /* 画像サイズセット */
  @media (max-width: 639px) {
    content-visibility: auto;
    contain-intrinsic-size: 0 300px;
  }
}

/* Container Queries対応 */
.container {
  container-type: inline-size;
  container-name: main;
}

@container main (min-width: 700px) {
  .responsive-component {
    /* コンテナ幅に応じたスタイル */
  }
}
```

## 8. アクセシビリティ対応 ♿

```css
/* フォーカス制御 */
@media (max-width: 767px) {
  .focus-trap {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
  }
}

/* タッチターゲットサイズ */
@media (max-width: 767px) {
  .touch-target {
    min-height: 44px;
    min-width: 44px;
    padding: 12px;
  }
}