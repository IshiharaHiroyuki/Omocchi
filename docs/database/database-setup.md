# データベース環境セットアップ手順書

## 📋 目次
1. [前提条件とシステム要件](#前提条件とシステム要件)
2. [インストール手順](#インストール手順)
3. [データベースの作成と設定](#データベースの作成と設定)
4. [初期データの投入](#初期データの投入)
5. [アクセス権限の設定](#アクセス権限の設定)
6. [動作確認](#動作確認)
7. [トラブルシューティング](#トラブルシューティング)
8. [バックアップ/リストア設定](#バックアップリストア設定)

## 💻 前提条件とシステム要件

### システム要件
- **CPU**: 4コア以上推奨
- **メモリ**: 16GB以上推奨（最小8GB）
- **ストレージ**: SSD 100GB以上（バックアップ領域含む）
- **OS**: Linux/Windows/macOS（Docker使用時）

### 必要なソフトウェア
- Docker Engine 20.10.x以上
- Docker Compose 2.x以上
- Node.js 18.x以上（マイグレーションツール用）

## 🛠️ インストール手順

### Docker Compose設定
以下の内容で`docker-compose.yml`を作成：

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    container_name: faq_postgres
    environment:
      POSTGRES_DB: faq_system
      POSTGRES_USER: faq_user
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    networks:
      - faq_network

  elasticsearch:
    image: elasticsearch:7.17.9
    container_name: faq_elasticsearch
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - faq_network

  redis:
    image: redis:6
    container_name: faq_redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    networks:
      - faq_network

volumes:
  postgres_data:
  elasticsearch_data:
  redis_data:

networks:
  faq_network:
    driver: bridge
```

### 環境変数の設定
`.env`ファイルを作成：

```env
POSTGRES_PASSWORD=your_secure_password
REDIS_PASSWORD=your_secure_redis_password
```

### コンテナの起動
```bash
# コンテナのビルドと起動
docker-compose up -d

# ログの確認
docker-compose logs -f
```

## 🗄️ データベースの作成と設定

### PostgreSQLデータベース初期化
`init-scripts/01-init.sql`を作成：

```sql
-- スキーマ作成
CREATE SCHEMA faq;

-- 拡張機能のインストール
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- テーブル作成
CREATE TABLE faq.users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- インデックス作成
CREATE INDEX idx_users_email ON faq.users(email);
```

### Elasticsearchインデックス設定
```bash
# FAQインデックスの作成
curl -X PUT "localhost:9200/faq" -H "Content-Type: application/json" -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "kuromoji_tokenizer",
          "filter": ["kuromoji_baseform", "lowercase"]
        }
      }
    }
  }
}'
```

### Redisの設定
```bash
# Redisの設定確認
docker-compose exec redis redis-cli -a ${REDIS_PASSWORD} CONFIG GET maxmemory-policy
```

## 📥 初期データの投入

### マイグレーションスクリプトの作成
`migrations/seed.js`を作成：

```javascript
const { Pool } = require('pg');
const fs = require('fs').promises;

const pool = new Pool({
  user: 'faq_user',
  password: process.env.POSTGRES_PASSWORD,
  host: 'localhost',
  database: 'faq_system',
  port: 5432,
});

async function seedDatabase() {
  try {
    // サンプルデータの読み込み
    const data = await fs.readFile('./seed-data.json', 'utf8');
    const seedData = JSON.parse(data);

    // データ投入
    for (const item of seedData) {
      await pool.query(
        'INSERT INTO faq.users (email, password_hash) VALUES ($1, $2)',
        [item.email, item.password_hash]
      );
    }

    console.log('Seed data inserted successfully');
  } catch (err) {
    console.error('Error seeding database:', err);
    throw err;
  } finally {
    await pool.end();
  }
}

seedDatabase();
```

### 初期データの実行
```bash
# 環境変数の設定
export POSTGRES_PASSWORD=your_secure_password

# マイグレーションの実行
node migrations/seed.js
```

## 🔒 アクセス権限の設定

### PostgreSQLユーザー権限
```sql
-- 読み取り専用ユーザーの作成
CREATE ROLE readonly_user WITH LOGIN PASSWORD 'readonly_password';
GRANT USAGE ON SCHEMA faq TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA faq TO readonly_user;

-- アプリケーション用ユーザーの作成
CREATE ROLE app_user WITH LOGIN PASSWORD 'app_password';
GRANT USAGE, CREATE ON SCHEMA faq TO app_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA faq TO app_user;
```

### Elasticsearchロール設定
```bash
# セキュリティ設定の追加
curl -X PUT "localhost:9200/_security/role/faq_read" -H "Content-Type: application/json" -d'
{
  "indices": [
    {
      "names": ["faq"],
      "privileges": ["read"]
    }
  ]
}'
```

## ✅ 動作確認

### PostgreSQL接続テスト
```bash
# psqlでの接続確認
docker-compose exec postgres psql -U faq_user -d faq_system

# テストクエリの実行
SELECT COUNT(*) FROM faq.users;
```

### Elasticsearch動作確認
```bash
# クラスタヘルスチェック
curl -X GET "localhost:9200/_cluster/health?pretty"

# インデックス確認
curl -X GET "localhost:9200/_cat/indices?v"
```

### Redis動作確認
```bash
# Redisへの接続確認
docker-compose exec redis redis-cli -a ${REDIS_PASSWORD} ping
```

## ❗ トラブルシューティング

### よくある問題と解決方法

1. **PostgreSQLコンテナが起動しない**
```bash
# ログの確認
docker-compose logs postgres

# データボリュームのクリア（注意：データが削除されます）
docker-compose down -v
docker-compose up -d
```

2. **Elasticsearchのメモリエラー**
```bash
# システムの設定確認
sysctl -w vm.max_map_count=262144

# Elasticsearchの再起動
docker-compose restart elasticsearch
```

3. **Redisの接続エラー**
```bash
# Redisコンテナの状態確認
docker-compose ps redis

# 設定の確認
docker-compose exec redis redis-cli -a ${REDIS_PASSWORD} INFO
```

## 💾 バックアップ/リストア設定

### PostgreSQLバックアップ設定

1. **定期バックアップスクリプト**
`scripts/backup-postgres.sh`を作成：

```bash
#!/bin/bash
BACKUP_DIR="/backup/postgres"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="faq_system_${TIMESTAMP}.dump"

# バックアップの実行
docker-compose exec -T postgres pg_dump -U faq_user faq_system > "${BACKUP_DIR}/${BACKUP_FILE}"

# 30日以上前のバックアップを削除
find ${BACKUP_DIR} -name "faq_system_*.dump" -mtime +30 -delete
```

2. **リストアスクリプト**
`scripts/restore-postgres.sh`を作成：

```bash
#!/bin/bash
BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
  echo "Usage: $0 <backup-file>"
  exit 1
fi

# リストアの実行
docker-compose exec -T postgres psql -U faq_user faq_system < "${BACKUP_FILE}"
```

### Elasticsearchスナップショット設定

1. **スナップショットリポジトリの登録**
```bash
# リポジトリの登録
curl -X PUT "localhost:9200/_snapshot/faq_backup" -H "Content-Type: application/json" -d'
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/backup"
  }
}'
```

2. **スナップショットの作成**
```bash
# スナップショットの実行
curl -X PUT "localhost:9200/_snapshot/faq_backup/snapshot_1?wait_for_completion=true"
```

### Redisバックアップ設定

1. **Redis設定ファイルの更新**
```conf
dir /data
dbfilename dump.rdb
save 900 1
save 300 10
save 60 10000
```

2. **バックアップスクリプト**
`scripts/backup-redis.sh`を作成：

```bash
#!/bin/bash
BACKUP_DIR="/backup/redis"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# BAKEコマンドの実行
docker-compose exec -T redis redis-cli -a ${REDIS_PASSWORD} SAVE

# バックアップファイルのコピー
docker cp faq_redis:/data/dump.rdb "${BACKUP_DIR}/dump_${TIMESTAMP}.rdb"

# 古いバックアップの削除
find ${BACKUP_DIR} -name "dump_*.rdb" -mtime +30 -delete
```

### Cronジョブの設定
```bash
# バックアップの自動化
0 1 * * * /path/to/scripts/backup-postgres.sh
0 2 * * * /path/to/scripts/backup-redis.sh
0 3 * * * curl -X PUT "localhost:9200/_snapshot/faq_backup/snapshot_$(date +\%Y\%m\%d)"
```

### 注意事項
- バックアップは必ず別ストレージに保存すること
- バックアップのリストアテストを定期的に実施すること
- バックアップログを定期的に確認すること
- クリティカルなデータは複数の場所にバックアップを保管すること