# Y-Hakua SNS - 統合開発環境

Google Classroomと連携した学習SNSアプリケーション「Y-Hakua SNS」の完全な統合開発環境です。このプロジェクトには、既存の機能に加えて、実装ガイドで提案された高優先度機能が統合されています。

## 🚀 新機能

### 1. 学習記録・進捗管理
- 日々の学習時間を記録
- 科目別の学習時間トラッキング
- グラフによる学習進捗の可視化

### 2. ノート共有・コラボレーション
- Markdown形式のノート作成・編集
- ノートの公開/非公開設定
- タグ付けと検索機能
- 「いいね」機能による評価

### 3. 課題提出状況の可視化
- Google Classroomの課題を一元管理
- 課題のステータス管理（未着手/進行中/完了）
- 期限リマインダー機能

### 4. 学習仲間マッチング
- 学習グループの作成・検索
- 同じコースを履修している仲間との繋がり
- グループチャット機能

### 5. ゲーミフィケーション
- ポイント・レベルシステム
- バッジ・アチーブメント
- 連続学習日数のストリーク表示
- リーダーボード

## 📋 システム要件

- Docker & Docker Compose
- Node.js 20+ (ローカル開発の場合)
- MongoDB (Docker内で自動起動)
- Redis (Docker内で自動起動)

## 🛠️ セットアップ

### 1. リポジトリのクローン

```bash
cd y-hakua-integrated
```

### 2. 環境変数の設定

```bash
# .env ファイルをコピー
cp .env.example .env

# .env ファイルを編集して、Google OAuth認証情報を設定
# GOOGLE_CLIENT_ID と GOOGLE_CLIENT_SECRET を入力
```

### 3. Docker Composeで起動

```bash
# すべてのサービスを起動
docker-compose up -d

# ログを確認
docker-compose logs -f

# 特定のサービスのログを確認
docker-compose logs -f backend
docker-compose logs -f frontend
```

### 4. アプリケーションへのアクセス

- **フロントエンド**: http://localhost:3000
- **バックエンド API**: http://localhost:8800
- **MongoDB**: mongodb://localhost:27017 (ユーザー: root, パスワード: password)
- **Redis**: redis://localhost:6379

## 📁 プロジェクト構造

```
y-hakua-integrated/
├── frontend/                    # React フロントエンド
│   ├── src/
│   │   ├── components/         # Reactコンポーネント
│   │   ├── pages/              # ページコンポーネント
│   │   ├── state/              # 状態管理
│   │   └── App.js
│   ├── Dockerfile
│   └── package.json
├── backend/                     # Express バックエンド
│   ├── models/                 # Mongooseモデル
│   │   ├── StudyLog.js         # 学習記録
│   │   ├── Note.js             # ノート
│   │   ├── UserAssignmentStatus.js  # 課題ステータス
│   │   ├── StudyGroup.js       # 学習グループ
│   │   └── UserAchievement.js  # ゲーミフィケーション
│   ├── routes/                 # APIルート
│   │   ├── studylogs.js        # 学習記録API
│   │   ├── notes.js            # ノートAPI
│   │   ├── studygroups.js      # 学習グループAPI
│   │   ├── gamification.js     # ゲーミフィケーションAPI
│   │   └── ...（既存ルート）
│   ├── controllers/            # ビジネスロジック
│   ├── middleware/             # ミドルウェア
│   ├── config/                 # 設定ファイル
│   ├── Dockerfile
│   ├── server.js               # メインサーバーファイル
│   └── package.json
├── docker-compose.yml          # Docker Compose設定
├── .env                        # 環境変数
├── .env.example                # 環境変数テンプレート
└── README.md
```

## 🔌 API エンドポイント

### 学習記録 API (`/api/studylogs`)

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| POST | `/` | 学習記録を作成 |
| GET | `/` | 学習記録一覧を取得 |
| GET | `/:id` | 特定の学習記録を取得 |
| PUT | `/:id` | 学習記録を更新 |
| DELETE | `/:id` | 学習記録を削除 |

**リクエスト例:**
```bash
curl -X POST http://localhost:8800/api/studylogs \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "数学",
    "durationMinutes": 60,
    "date": "2026-02-01",
    "notes": "微分積分の復習"
  }'
```

### ノート API (`/api/notes`)

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| POST | `/` | ノートを作成 |
| GET | `/` | ノート一覧を取得 |
| GET | `/:id` | 特定のノートを取得 |
| PUT | `/:id` | ノートを更新 |
| DELETE | `/:id` | ノートを削除 |
| PUT | `/like/:id` | ノートに「いいね」 |
| PUT | `/unlike/:id` | 「いいね」を取り消し |

### 学習グループ API (`/api/studygroups`)

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| POST | `/` | グループを作成 |
| GET | `/` | グループ一覧を取得 |
| GET | `/:id` | 特定のグループを取得 |
| PUT | `/join/:id` | グループに参加 |
| PUT | `/leave/:id` | グループから脱退 |
| PUT | `/:id` | グループを更新（管理者のみ） |
| DELETE | `/:id` | グループを削除（管理者のみ） |

### ゲーミフィケーション API (`/api/gamification`)

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| GET | `/me` | ユーザーの成績を取得 |
| GET | `/leaderboard` | リーダーボードを取得 |
| GET | `/:userId` | 特定ユーザーの成績を取得 |

## 🔐 認証

すべてのAPIエンドポイントはJWT認証が必要です。リクエストヘッダーに以下を含めてください：

```
Authorization: Bearer YOUR_JWT_TOKEN
```

## 📊 データベーススキーマ

### StudyLog
```javascript
{
  userId: ObjectId,
  subject: String,
  durationMinutes: Number,
  date: Date,
  notes: String,
  courseId: ObjectId,
  assignmentId: String,
  createdAt: Date,
  updatedAt: Date
}
```

### Note
```javascript
{
  userId: ObjectId,
  courseId: ObjectId,
  title: String,
  content: String,
  isPublic: Boolean,
  tags: [String],
  likes: [ObjectId],
  comments: [ObjectId],
  createdAt: Date,
  updatedAt: Date
}
```

### StudyGroup
```javascript
{
  name: String,
  description: String,
  adminId: ObjectId,
  members: [ObjectId],
  courseId: ObjectId,
  topics: [String],
  chatRoomId: ObjectId,
  createdAt: Date,
  updatedAt: Date
}
```

### UserAchievement
```javascript
{
  userId: ObjectId,
  totalPoints: Number,
  level: Number,
  badges: [String],
  lastActivityDate: Date,
  currentStreak: Number,
  longestStreak: Number,
  createdAt: Date,
  updatedAt: Date
}
```

## 🧪 テスト

### バックエンドのテスト

```bash
# バックエンドコンテナに入る
docker-compose exec backend sh

# テストを実行（テストスクリプトが設定されている場合）
npm test
```

### APIのテスト

```bash
# ヘルスチェック
curl http://localhost:8800/health

# 学習記録の取得
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:8800/api/studylogs
```

## 🐛 トラブルシューティング

### MongoDB接続エラー

```bash
# MongoDBコンテナの状態を確認
docker-compose ps

# MongoDBコンテナのログを確認
docker-compose logs mongodb

# コンテナを再起動
docker-compose restart mongodb
```

### ポート競合

別のアプリケーションがポート3000、8800、27017を使用している場合は、`docker-compose.yml`のポート設定を変更してください。

```yaml
ports:
  - "3001:3000"  # フロントエンド
  - "8801:8800"  # バックエンド
  - "27018:27017"  # MongoDB
```

### ビルドエラー

```bash
# キャッシュをクリアして再ビルド
docker-compose build --no-cache

# すべてのコンテナを削除して再起動
docker-compose down -v
docker-compose up -d
```

## 📝 開発ワークフロー

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

### Docker開発

```bash
# ホットリロード有効で起動
docker-compose up -d

# コード変更時は自動的にリロード
```

## 🚀 デプロイ

### 本番環境への準備

1. `.env`ファイルを本番用に設定
2. Google OAuth認証情報を設定
3. Cloudinary認証情報を設定（オプション）
4. JWT_SECRETを変更

```bash
# 本番ビルド
docker-compose -f docker-compose.prod.yml build

# 本番起動
docker-compose -f docker-compose.prod.yml up -d
```

## 📚 参考資料

- [実装ガイド](./implementation_guide.md)
- [既存構造分析](./existing_structure_analysis.md)
- [React ドキュメント](https://react.dev)
- [Express ドキュメント](https://expressjs.com)
- [MongoDB ドキュメント](https://docs.mongodb.com)
- [Socket.io ドキュメント](https://socket.io)

## 🤝 貢献

プルリクエストを歓迎します。大きな変更の場合は、まずissueを開いて変更内容を議論してください。

## 📄 ライセンス

このプロジェクトはISCライセンスの下で公開されています。

## 📧 サポート

問題が発生した場合は、GitHubのissueを作成してください。

---

**作成日**: 2026年2月1日
**バージョン**: 1.0.0
**ステータス**: 開発中
