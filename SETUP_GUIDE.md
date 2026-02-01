# Y-Hakua SNS 統合環境 セットアップガイド

このガイドは、Docker Composeを使用してY-Hakua SNS統合環境をセットアップするステップバイステップの手順を提供します。

## 前提条件

- Docker Desktop がインストールされている（Windows/Mac）または Docker + Docker Compose（Linux）
- Git がインストールされている
- テキストエディタ（VS Code推奨）

## ステップ1: リポジトリの準備

```bash
# プロジェクトディレクトリに移動
cd y-hakua-integrated

# ディレクトリ構造を確認
ls -la
# 以下が表示されるはず:
# - backend/
# - frontend/
# - docker-compose.yml
# - .env
# - .env.example
# - README.md
```

## ステップ2: 環境変数の設定

### 2.1 .env ファイルの確認

```bash
cat .env
```

### 2.2 Google OAuth 認証情報の取得（オプションだが推奨）

Google Classroomと連携するには、Google Cloud Consoleで認証情報を取得する必要があります。

1. [Google Cloud Console](https://console.cloud.google.com) にアクセス
2. 新しいプロジェクトを作成
3. Google Classroom API を有効化
4. OAuth 2.0 クライアント ID を作成
5. 認可済みリダイレクト URI に以下を追加:
   - `http://localhost:3000/auth/google/callback`
   - `http://localhost:8800/auth/google/callback`

### 2.3 .env ファイルを編集

```bash
# テキストエディタで .env を開く
nano .env
# または
code .env

# 以下の項目を編集:
GOOGLE_CLIENT_ID=your_client_id_here
GOOGLE_CLIENT_SECRET=your_client_secret_here
REACT_APP_GOOGLE_CLIENT_ID=your_client_id_here
```

### 2.4 Cloudinary 設定（オプション、画像アップロード用）

1. [Cloudinary](https://cloudinary.com) でアカウント作成
2. ダッシュボードから認証情報を取得
3. .env に追加:

```bash
CLOUDINARY_NAME=your_cloudinary_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

## ステップ3: Docker イメージのビルド

```bash
# すべてのイメージをビルド
docker-compose build

# または、キャッシュをクリアしてビルド
docker-compose build --no-cache
```

## ステップ4: サービスの起動

```bash
# バックグラウンドで起動
docker-compose up -d

# または、ログを表示しながら起動
docker-compose up
```

## ステップ5: サービスの確認

```bash
# 実行中のコンテナを確認
docker-compose ps

# 以下が表示されるはず:
# NAME                COMMAND                  SERVICE             STATUS
# y-hakua-backend     "npm start"              backend             Up
# y-hakua-frontend    "npm start"              frontend            Up
# y-hakua-mongodb     "mongod"                 mongodb             Up
# y-hakua-redis       "redis-server"           redis               Up
```

## ステップ6: ヘルスチェック

```bash
# バックエンド API のヘルスチェック
curl http://localhost:8800/health

# 以下が返されるはず:
# {"status":"OK","uptime":123.45,"timestamp":1234567890}
```

## ステップ7: アプリケーションへのアクセス

ブラウザで以下のURLにアクセス:

- **フロントエンド**: http://localhost:3000
- **バックエンド API**: http://localhost:8800

## ステップ8: ログの確認

```bash
# すべてのサービスのログを表示
docker-compose logs -f

# 特定のサービスのログを表示
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mongodb
docker-compose logs -f redis

# 最新の50行を表示
docker-compose logs --tail=50
```

## ステップ9: データベースの初期化

### MongoDB への接続

```bash
# MongoDB コンテナに接続
docker-compose exec mongodb mongosh -u root -p password

# または、MongoDB Compass を使用
# URI: mongodb://root:password@localhost:27017/y-hakua?authSource=admin
```

### テストデータの挿入（オプション）

```bash
# backend コンテナに入る
docker-compose exec backend sh

# Node.js REPL を起動
node

# テストデータを挿入するスクリプトを実行
# （別途スクリプトが必要）
```

## ステップ10: 開発の開始

### コード変更時の自動リロード

Docker Compose で起動している場合、コード変更時に自動的にリロードされます。

```bash
# フロントエンドのコードを編集
code frontend/src/App.js

# バックエンドのコードを編集
code backend/server.js

# 変更は自動的に反映されます
```

### ローカル開発（Docker不使用）

```bash
# バックエンド
cd backend
npm install
npm start

# フロントエンド（別ターミナル）
cd frontend
npm install
npm start
```

## トラブルシューティング

### ポート競合エラー

```
Error: listen EADDRINUSE: address already in use :::3000
```

**解決方法:**

```bash
# ポート3000を使用しているプロセスを確認
lsof -i :3000

# または、docker-compose.yml でポートを変更
# ports:
#   - "3001:3000"
```

### MongoDB 接続エラー

```
MongooseError: Cannot connect to MongoDB
```

**解決方法:**

```bash
# MongoDB コンテナの状態を確認
docker-compose ps mongodb

# MongoDB コンテナを再起動
docker-compose restart mongodb

# ログを確認
docker-compose logs mongodb
```

### メモリ不足エラー

```
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
```

**解決方法:**

```bash
# Docker Desktop のメモリ設定を増やす
# Settings > Resources > Memory を 4GB 以上に設定

# または、Node.js のヒープサイズを増やす
# docker-compose.yml で以下を追加:
# environment:
#   NODE_OPTIONS: --max-old-space-size=2048
```

### npm install エラー

```bash
# キャッシュをクリア
docker-compose exec backend npm cache clean --force

# 再度ビルド
docker-compose build --no-cache
```

## コンテナの管理

### コンテナの停止

```bash
# すべてのコンテナを停止
docker-compose stop

# 特定のコンテナを停止
docker-compose stop backend
```

### コンテナの再起動

```bash
# すべてのコンテナを再起動
docker-compose restart

# 特定のコンテナを再起動
docker-compose restart backend
```

### コンテナの削除

```bash
# 停止中のコンテナを削除
docker-compose rm

# 実行中のコンテナを削除
docker-compose rm -f

# ボリュームも削除
docker-compose down -v
```

## 本番環境への準備

### 環境変数の本番設定

```bash
# .env.production を作成
cp .env .env.production

# 本番用の値を設定
nano .env.production

# 重要な設定:
# - NODE_ENV=production
# - JWT_SECRET を強力なものに変更
# - MONGO_URI を本番 MongoDB に変更
# - REDIS_URL を本番 Redis に変更
```

### Docker イメージのビルド

```bash
# 本番用イメージをビルド
docker build -t y-hakua-backend:1.0.0 ./backend
docker build -t y-hakua-frontend:1.0.0 ./frontend

# イメージをレジストリにプッシュ
docker tag y-hakua-backend:1.0.0 your-registry/y-hakua-backend:1.0.0
docker push your-registry/y-hakua-backend:1.0.0
```

## 次のステップ

1. [README.md](./README.md) を読んで、プロジェクト構造を理解する
2. [実装ガイド](./implementation_guide.md) を読んで、新機能の詳細を理解する
3. API ドキュメントを確認する
4. テストコードを作成する
5. 本番環境にデプロイする

## サポート

問題が発生した場合:

1. ログを確認: `docker-compose logs -f`
2. [トラブルシューティング](#トラブルシューティング) セクションを参照
3. GitHub Issues で報告

---

**最終更新**: 2026年2月1日
