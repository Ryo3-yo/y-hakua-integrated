# Y-Hakua SNS データベース管理ガイド

このガイドは、Y-Hakua SNS のデータベース（MongoDB）の管理と初期化方法を説明します。

## 📊 データベース構成

### MongoDB コンテナ

- **ホスト**: localhost
- **ポート**: 27017
- **ユーザー**: root
- **パスワード**: password
- **データベース**: y-hakua
- **認証ソース**: admin

### 接続文字列

```
mongodb://root:password@localhost:27017/y-hakua?authSource=admin
```

## 🔧 MongoDB への接続

### mongosh を使用

```bash
# MongoDB コンテナに接続
docker-compose exec mongodb mongosh -u root -p password

# y-hakua データベースに切り替え
use y-hakua

# コレクション一覧を表示
show collections

# ドキュメント数を確認
db.users.countDocuments()
```

### MongoDB Compass を使用（GUI）

1. [MongoDB Compass](https://www.mongodb.com/products/compass) をダウンロード
2. 以下の情報で接続:
   - **URI**: `mongodb://root:password@localhost:27017/y-hakua?authSource=admin`
   - または個別に入力:
     - Host: localhost
     - Port: 27017
     - Username: root
     - Password: password
     - Authentication Database: admin

## 📝 初期データの挿入

### 方法1: スクリプトを使用（推奨）

```bash
# backend コンテナで初期化スクリプトを実行
docker-compose exec backend node scripts/seedData.js
```

**出力例:**
```
✓ MongoDB に接続しました
✓ 3 人のテストユーザーを作成しました
✓ 2 個のテストコースを作成しました
✓ 10 件の学習記録を作成しました
✓ 2 件のノートを作成しました
✓ 2 個の学習グループを作成しました
✓ 3 件のアチーブメントを作成しました

✅ すべての初期データが正常に作成されました！
```

### 方法2: mongosh で手動挿入

```javascript
// MongoDB に接続後

// ユーザーを挿入
db.users.insertOne({
  name: "テストユーザー",
  email: "test@example.com",
  googleId: "google_id_123",
  avatar: "https://via.placeholder.com/150"
})

// 学習記録を挿入
db.studylogs.insertOne({
  userId: ObjectId("..."),
  subject: "数学",
  durationMinutes: 60,
  date: new Date(),
  notes: "微分積分の復習"
})

// 学習グループを挿入
db.studygroups.insertOne({
  name: "Python 学習グループ",
  description: "Python を一緒に勉強するグループ",
  adminId: ObjectId("..."),
  members: [ObjectId("...")],
  topics: ["Python", "プログラミング"]
})
```

## 📋 コレクション構造

### Users Collection

```javascript
{
  _id: ObjectId,
  name: String,
  email: String,
  googleId: String,
  avatar: String,
  createdAt: Date,
  updatedAt: Date
}
```

### StudyLogs Collection

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: User),
  subject: String,
  durationMinutes: Number,
  date: Date,
  notes: String,
  courseId: ObjectId (ref: Course),
  createdAt: Date,
  updatedAt: Date
}
```

### Notes Collection

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: User),
  courseId: ObjectId (ref: Course),
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

### StudyGroups Collection

```javascript
{
  _id: ObjectId,
  name: String,
  description: String,
  adminId: ObjectId (ref: User),
  members: [ObjectId],
  courseId: ObjectId (ref: Course),
  topics: [String],
  chatRoomId: ObjectId (ref: Conversation),
  createdAt: Date,
  updatedAt: Date
}
```

### UserAchievements Collection

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: User),
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

## 🔍 よく使うクエリ

### ユーザー関連

```javascript
// すべてのユーザーを取得
db.users.find()

// 特定のユーザーを取得
db.users.findOne({ email: "test@example.com" })

// ユーザー数をカウント
db.users.countDocuments()

// ユーザーを更新
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { name: "新しい名前" } }
)

// ユーザーを削除
db.users.deleteOne({ _id: ObjectId("...") })
```

### 学習記録関連

```javascript
// 特定ユーザーの学習記録を取得
db.studylogs.find({ userId: ObjectId("...") })

// 日付範囲で学習記録を取得
db.studylogs.find({
  date: {
    $gte: new Date("2026-02-01"),
    $lte: new Date("2026-02-28")
  }
})

// 科目別に学習記録を集計
db.studylogs.aggregate([
  {
    $group: {
      _id: "$subject",
      totalMinutes: { $sum: "$durationMinutes" },
      count: { $sum: 1 }
    }
  }
])

// ユーザーの総学習時間を計算
db.studylogs.aggregate([
  { $match: { userId: ObjectId("...") } },
  { $group: { _id: null, totalMinutes: { $sum: "$durationMinutes" } } }
])
```

### ノート関連

```javascript
// 公開ノートを取得
db.notes.find({ isPublic: true })

// タグで検索
db.notes.find({ tags: "Python" })

// いいね数でソート
db.notes.find().sort({ "likes": -1 }).limit(10)
```

### 学習グループ関連

```javascript
// すべてのグループを取得
db.studygroups.find()

// メンバー数でグループを検索
db.studygroups.find({ members: ObjectId("...") })

// グループの詳細情報を取得（populate）
db.studygroups.aggregate([
  { $match: { _id: ObjectId("...") } },
  { $lookup: { from: "users", localField: "adminId", foreignField: "_id", as: "admin" } },
  { $lookup: { from: "users", localField: "members", foreignField: "_id", as: "memberDetails" } }
])
```

## 🔄 バックアップとリストア

### バックアップ

```bash
# MongoDB データベースをバックアップ
docker-compose exec mongodb mongodump \
  -u root \
  -p password \
  --authenticationDatabase admin \
  --out /backup

# バックアップをホストにコピー
docker cp y-hakua-mongodb:/backup ./mongodb_backup
```

### リストア

```bash
# バックアップからリストア
docker-compose exec mongodb mongorestore \
  -u root \
  -p password \
  --authenticationDatabase admin \
  /backup
```

## 🧹 データベースのクリーンアップ

### すべてのコレクションを削除

```bash
docker-compose exec mongodb mongosh -u root -p password << EOF
use y-hakua
db.users.deleteMany({})
db.studylogs.deleteMany({})
db.notes.deleteMany({})
db.studygroups.deleteMany({})
db.userachievements.deleteMany({})
db.courses.deleteMany({})
db.conversations.deleteMany({})
db.messages.deleteMany({})
db.posts.deleteMany({})
db.comments.deleteMany({})
db.notifications.deleteMany({})
db.hashtags.deleteMany({})
EOF
```

### 特定のコレクションを削除

```bash
docker-compose exec mongodb mongosh -u root -p password << EOF
use y-hakua
db.studylogs.deleteMany({})
EOF
```

## 📊 インデックス管理

### インデックスを作成

```javascript
// ユーザーのメールアドレスに一意インデックスを作成
db.users.createIndex({ email: 1 }, { unique: true })

// 学習記録の日付にインデックスを作成
db.studylogs.createIndex({ date: -1 })

// 複合インデックスを作成
db.studylogs.createIndex({ userId: 1, date: -1 })
```

### インデックスを確認

```javascript
// すべてのインデックスを表示
db.users.getIndexes()
```

### インデックスを削除

```javascript
// 特定のインデックスを削除
db.users.dropIndex("email_1")

// すべてのインデックスを削除（_id を除く）
db.users.dropIndexes()
```

## 🔐 セキュリティ

### ユーザーを作成

```bash
docker-compose exec mongodb mongosh -u root -p password << EOF
use admin
db.createUser({
  user: "y-hakua-user",
  pwd: "strong_password_here",
  roles: [
    { role: "readWrite", db: "y-hakua" }
  ]
})
EOF
```

### パスワードを変更

```bash
docker-compose exec mongodb mongosh -u root -p password << EOF
use admin
db.changeUserPassword("y-hakua-user", "new_password")
EOF
```

## 📈 パフォーマンス最適化

### クエリプランを分析

```javascript
// クエリの実行計画を確認
db.studylogs.find({ userId: ObjectId("...") }).explain("executionStats")

// インデックスが使用されているか確認
db.studylogs.find({ userId: ObjectId("...") }).hint({ userId: 1 }).explain()
```

### スローログを有効化

```javascript
// スローログを有効化（100ms以上のクエリをログ）
db.setProfilingLevel(1, { slowms: 100 })

// スローログを確認
db.system.profile.find().sort({ ts: -1 }).limit(10).pretty()

// スローログを無効化
db.setProfilingLevel(0)
```

## 🆘 トラブルシューティング

### MongoDB に接続できない

```bash
# MongoDB コンテナの状態を確認
docker-compose ps mongodb

# MongoDB コンテナのログを確認
docker-compose logs mongodb

# コンテナを再起動
docker-compose restart mongodb
```

### ディスク容量が足りない

```bash
# データベースサイズを確認
docker-compose exec mongodb mongosh -u root -p password << EOF
use y-hakua
db.stats()
EOF

# 不要なコレクションを削除
db.collection_name.drop()
```

### パフォーマンスが低下

```javascript
// インデックスを再構築
db.collection_name.reIndex()

// コレクションを最適化
db.collection_name.compact()
```

## 📚 参考資料

- [MongoDB ドキュメント](https://docs.mongodb.com)
- [Mongoose ドキュメント](https://mongoosejs.com)
- [MongoDB Compass ドキュメント](https://docs.mongodb.com/compass)

---

**最終更新**: 2026年2月1日
