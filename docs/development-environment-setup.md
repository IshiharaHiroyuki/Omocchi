# 開発環境構築手順書

## 📋 目次
1. [開発環境要件](#開発環境要件)
2. [必要なソフトウェアとツール](#必要なソフトウェアとツール)
3. [依存関係の導入手順](#依存関係の導入手順)
4. [リポジトリのセットアップ](#リポジトリのセットアップ)
5. [開発用サービスの起動](#開発用サービスの起動)
6. [デバッグ環境の設定](#デバッグ環境の設定)
7. [テスト実行手順](#テスト実行手順)
8. [コーディング規約](#コーディング規約)
9. [トラブルシューティング](#トラブルシューティング)

## 🖥️ 開発環境要件

### システム要件
- **OS**
  - Windows 10/11
  - macOS Monterey以降
  - Ubuntu 20.04 LTS以降
- **メモリ**: 16GB以上推奨（最小8GB）
- **ストレージ**: 256GB以上のSSD推奨
- **CPU**: Intel Core i5/AMD Ryzen 5以上推奨
- **ネットワーク**: ブロードバンドインターネット接続

### 推奨開発ツール
- **IDE**: Visual Studio Code
- **ブラウザ**: Chrome最新版（開発者ツール使用）
- **バージョン管理**: Git 2.x以上

## 🛠️ 必要なソフトウェアとツール

### 基本開発環境
1. **Node.js**
   - バージョン: 18.x LTS
   - [ダウンロードリンク](https://nodejs.org/)

2. **Docker**
   - バージョン: 最新安定版
   - [ダウンロードリンク](https://www.docker.com/products/docker-desktop)

3. **Git**
   - バージョン: 2.x以上
   - [ダウンロードリンク](https://git-scm.com/downloads)

### データベース関連
1. **PostgreSQL**
   - バージョン: 14.x
   - Docker経由でのインストールを推奨

2. **Elasticsearch**
   - バージョン: 7.x
   - Docker経由でのインストールを推奨

3. **Redis**
   - バージョン: 6.x
   - Docker経由でのインストールを推奨

## 📦 依存関係の導入手順

### フロントエンド（React + TypeScript）
```bash
# プロジェクトルートディレクトリで実行
cd frontend
npm install

# 主要な依存関係
npm install react@18.x
npm install @types/react@18.x
npm install typescript@4.x
npm install @types/node@18.x
npm install react-router-dom@6.x
npm install @reduxjs/toolkit react-redux
```

### バックエンド（Express.js + TypeScript）
```bash
# プロジェクトルートディレクトリで実行
cd backend
npm install

# 主要な依存関係
npm install express@4.x
npm install @types/express@4.x
npm install typescript@4.x
npm install pg@8.x
npm install @elastic/elasticsearch@7.x
npm install redis@4.x
```

## 🔧 リポジトリのセットアップ

### リポジトリのクローン
```bash
git clone <repository-url>
cd <project-directory>
```

### ブランチ戦略
- `main`: 本番環境用のブランチ
- `develop`: 開発環境用のブランチ
- `feature/*`: 機能開発用のブランチ
- `fix/*`: バグ修正用のブランチ
- `release/*`: リリース準備用のブランチ

### 初期設定
```bash
# Git設定
git config user.name "Your Name"
git config user.email "your.email@example.com"

# 開発ブランチの作成
git checkout -b develop origin/develop
```

## 🚀 開発用サービスの起動

### Dockerコンテナの起動
```bash
# プロジェクトルートディレクトリで実行
docker-compose up -d
```

### フロントエンド開発サーバーの起動
```bash
cd frontend
npm run dev
```

### バックエンド開発サーバーの起動
```bash
cd backend
npm run dev
```

## 🐛 デバッグ環境の設定

### VSCode デバッグ設定
`.vscode/launch.json`を作成：
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/backend/dist/index.js",
      "preLaunchTask": "npm: build",
      "outFiles": ["${workspaceFolder}/backend/dist/**/*.js"],
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Frontend",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/frontend"
    }
  ]
}
```

## ✅ テスト実行手順

### ユニットテストの実行
```bash
# フロントエンドのテスト
cd frontend
npm test

# バックエンドのテスト
cd backend
npm test
```

### E2Eテストの実行
```bash
# Cypressテストの実行
npm run e2e
```

### カバレッジレポートの生成
```bash
# フロントエンドのカバレッジ
cd frontend
npm run test:coverage

# バックエンドのカバレッジ
cd backend
npm run test:coverage
```

## 📝 コーディング規約

詳細は `docs/coding-guidelines.md` を参照してください。

主要なポイント：
- TypeScriptを使用し、型定義を徹底する
- コンポーネントは関数コンポーネントを使用
- テストファーストの開発アプローチ
- コードレビュー前のセルフチェックリストの確認

## ❗ トラブルシューティング

### よくある問題と解決方法

1. **Dockerコンテナが起動しない**
```bash
# ログの確認
docker-compose logs

# コンテナの再ビルド
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

2. **npm installでエラー**
```bash
# キャッシュのクリア
npm cache clean --force

# package-lock.jsonの削除と再インストール
rm package-lock.json
npm install
```

3. **TypeScriptのコンパイルエラー**
```bash
# TypeScriptの設定確認
cat tsconfig.json

# 依存関係の型定義の再インストール
npm install @types/node @types/react @types/express --save-dev
```

### サポートリソース
- プロジェクトWiki
- チームSlackチャンネル
- 技術文書（`docs/`ディレクトリ）

### 注意事項
- 本番環境の認証情報は`.env`ファイルで管理し、Gitにはコミットしない
- データベースのマイグレーションは必ずバックアップを取ってから実行
- 新機能の開発は必ずfeatureブランチで行う