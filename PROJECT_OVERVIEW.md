# Y-Hakua SNS プロジェクト概要

## 🎯 プロジェクト目標

Y-Hakua SNS は、Google Classroom と連携した学習SNSアプリケーションです。学習者が所属するクラスルームの投稿を見ながら、同時に学習記録、ノート共有、学習仲間マッチング、ゲーミフィケーション機能を利用できます。

## 📚 主な特徴

### 既存機能
- **Google Classroom 連携** - クラスルームの課題・投稿を表示
- **ユーザー認証** - Google OAuth 2.0
- **投稿・メッセージング** - リアルタイムチャット
- **ファイルアップロード** - Cloudinary 統合

### 新機能（実装ガイドから）
1. **学習記録・進捗管理** - 学習時間の記録と可視化
2. **ノート共有・コラボレーション** - Markdown形式のノート作成・共有
3. **課題提出状況の可視化** - Google Classroom との連携
4. **学習仲間マッチング** - 学習グループの作成・検索
5. **ゲーミフィケーション** - ポイント・バッジ・リーダーボード

## 🏗️ アーキテクチャ

### システム構成図

```
┌─────────────────────────────────────────────────────────┐
│                    ユーザーブラウザ                        │
└────────────────────────┬────────────────────────────────┘
                         │
                    HTTP/WebSocket
                         │
        ┌────────────────┴────────────────┐
        │                                 │
┌───────▼──────────┐          ┌──────────▼──────────┐
│   フロントエンド    │          │   バックエンド API    │
│  (React 19.2.0)  │          │  (Express 5.1.0)   │
│  Port: 3000      │          │  Port: 8800        │
└───────┬──────────┘          └──────────┬──────────┘
        │                                 │
        └─────────────────┬───────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
    ┌───▼──────┐   ┌─────▼────┐   ┌──────▼────┐
    │ MongoDB  │   │  Redis   │   │ Google    │
    │ Port:    │   │ Port:    │   │ Classroom │
    │ 27017    │   │ 6379     │   │ API       │
    └──────────┘   └──────────┘   └───────────┘
```

### 技術スタック

| レイヤー | 技術 | バージョン |
|---------|------|-----------|
| フロントエンド | React | 19.2.0 |
| フロントエンド | Material-UI | 7.3.4 |
| フロントエンド | Socket.io Client | 4.7.5 |
| バックエンド | Express | 5.1.0 |
| バックエンド | Node.js | 20 |
| データベース | MongoDB | 7.0 |
| キャッシング | Redis | 7 |
| 認証 | Passport.js | 0.7.0 |
| 認証 | Google OAuth 2.0 | - |
| リアルタイム | Socket.io | 4.7.5 |
| ファイル保存 | Cloudinary | 1.41.3 |
| コンテナ化 | Docker | Latest |

## 📁 ディレクトリ構造

```
y-hakua-integrated/
├── frontend/                           # React フロントエンド
│   ├── src/
│   │   ├── components/
│   │   │   ├── StudyLogForm.js        # 学習記録フォーム
│   │   │   ├── StudyLogList.js        # 学習記録一覧
│   │   │   ├── GamificationDashboard.js
│   │   │   ├── NoteEditor.js          # ノートエディタ
│   │   │   ├── StudyGroupCard.js      # グループカード
│   │   │   └── ...（既存コンポーネント）
│   │   ├── pages/
│   │   │   ├── HomePage.js
│   │   │   ├── ClassroomPage.js
│   │   │   └── ...（既存ページ）
│   │   ├── state/
│   │   │   ├── StudyLogContext.js     # 学習記録状態管理
│   │   │   ├── NoteContext.js         # ノート状態管理
│   │   │   ├── StudyGroupContext.js   # グループ状態管理
│   │   │   ├── GamificationContext.js # ゲーミフィケーション状態管理
│   │   │   └── ...（既存コンテキスト）
│   │   ├── App.js
│   │   └── index.js
│   ├── Dockerfile
│   ├── package.json
│   └── .env.production
│
├── backend/                            # Express バックエンド
│   ├── models/
│   │   ├── StudyLog.js                # 学習記録モデル
│   │   ├── Note.js                    # ノートモデル
│   │   ├── UserAssignmentStatus.js    # 課題ステータスモデル
│   │   ├── StudyGroup.js              # グループモデル
│   │   ├── UserAchievement.js         # アチーブメントモデル
│   │   └── ...（既存モデル）
│   ├── routes/
│   │   ├── studylogs.js               # 学習記録API
│   │   ├── notes.js                   # ノートAPI
│   │   ├── studygroups.js             # グループAPI
│   │   ├── gamification.js            # ゲーミフィケーションAPI
│   │   └── ...（既存ルート）
│   ├── controllers/
│   ├── middleware/
│   ├── config/
│   ├── scripts/
│   │   └── seedData.js                # 初期データ挿入スクリプト
│   ├── Dockerfile
│   ├── server.js
│   ├── package.json
│   └── redisClient.js
│
├── docker-compose.yml                 # Docker Compose 設定
├── .env                               # 環境変数
├── .env.example                       # 環境変数テンプレート
│
├── README.md                          # プロジェクト概要
├── QUICK_START.md                     # クイックスタート
├── SETUP_GUIDE.md                     # セットアップガイド
├── INTEGRATION_GUIDE.md               # 統合ガイド
├── DATABASE_GUIDE.md                  # データベース管理
└── PROJECT_OVERVIEW.md                # このファイル
```

## 🔌 API エンドポイント一覧

### 認証 API (`/api/auth`)
- `POST /login` - ログイン
- `POST /logout` - ログアウト
- `POST /refresh` - トークン更新

### ユーザー API (`/api/users`)
- `GET /` - ユーザー一覧取得
- `GET /:id` - ユーザー詳細取得
- `PUT /:id` - ユーザー更新
- `DELETE /:id` - ユーザー削除

### 投稿 API (`/api/posts`)
- `POST /` - 投稿作成
- `GET /` - 投稿一覧取得
- `GET /:id` - 投稿詳細取得
- `PUT /:id` - 投稿更新
- `DELETE /:id` - 投稿削除

### Google Classroom API (`/api/classroom`)
- `GET /courses` - コース一覧取得
- `GET /courses/:id/assignments` - 課題一覧取得
- `GET /courses/:id/posts` - 投稿一覧取得

### 学習記録 API (`/api/studylogs`)
- `POST /` - 学習記録作成
- `GET /` - 学習記録一覧取得
- `GET /:id` - 学習記録詳細取得
- `PUT /:id` - 学習記録更新
- `DELETE /:id` - 学習記録削除

### ノート API (`/api/notes`)
- `POST /` - ノート作成
- `GET /` - ノート一覧取得
- `GET /:id` - ノート詳細取得
- `PUT /:id` - ノート更新
- `DELETE /:id` - ノート削除
- `PUT /like/:id` - ノートに「いいね」
- `PUT /unlike/:id` - 「いいね」取り消し

### 学習グループ API (`/api/studygroups`)
- `POST /` - グループ作成
- `GET /` - グループ一覧取得
- `GET /:id` - グループ詳細取得
- `PUT /join/:id` - グループに参加
- `PUT /leave/:id` - グループから脱退
- `PUT /:id` - グループ更新
- `DELETE /:id` - グループ削除

### ゲーミフィケーション API (`/api/gamification`)
- `GET /me` - ユーザーの成績取得
- `GET /leaderboard` - リーダーボード取得
- `GET /:userId` - ユーザーの成績取得

### その他 API
- `GET /api/messages` - メッセージ取得
- `GET /api/conversations` - 会話取得
- `GET /api/notifications` - 通知取得
- `GET /api/hashtags` - ハッシュタグ取得
- `POST /api/upload` - ファイルアップロード

## 🔐 認証フロー

```
1. ユーザーがGoogle OAuth でログイン
   ↓
2. バックエンドが Google から ユーザー情報を取得
   ↓
3. ユーザーが DB に存在しなければ作成
   ↓
4. JWT トークンを生成
   ↓
5. フロントエンドが localStorage にトークンを保存
   ↓
6. 以降のリクエストで Authorization ヘッダーにトークンを含める
```

## 📊 データフロー

### 学習記録の作成フロー

```
フロントエンド                    バックエンド              データベース
    │                              │                         │
    │ 1. StudyLogForm で入力       │                         │
    │    (科目、時間、日付など)     │                         │
    │                              │                         │
    │ 2. POST /api/studylogs       │                         │
    ├─────────────────────────────>│                         │
    │                              │ 3. StudyLog 作成        │
    │                              ├────────────────────────>│
    │                              │                         │ 4. DB に保存
    │                              │<────────────────────────┤
    │                              │ 5. UserAchievement 更新 │
    │                              ├────────────────────────>│
    │                              │                         │ 6. ポイント加算
    │                              │<────────────────────────┤
    │ 7. 成功レスポンス            │                         │
    │<─────────────────────────────┤                         │
    │                              │                         │
    │ 8. StudyLogList を更新       │                         │
    │    (新しい記録を表示)         │                         │
    │                              │                         │
```

## 🔄 リアルタイム機能

### Socket.io イベント

```javascript
// ユーザーがオンラインになった
socket.emit('addUser', userId);

// メッセージを送信
socket.emit('sendMessage', {
  senderId, senderName, receiverId, text, conversationId
});

// メッセージを既読
socket.emit('markAsRead', { conversationId, readerId, senderId });

// タイピング中
socket.emit('typing', { conversationId, userId, receiverId });

// タイピング停止
socket.emit('stopTyping', { conversationId, userId, receiverId });
```

## 📈 スケーラビリティ

### 現在の構成
- **単一インスタンス** - 開発環境向け
- **ローカルストレージ** - セッション管理

### 本番環境への拡張
- **複数インスタンス** - ロードバランサーで分散
- **Redis セッションストア** - セッション共有
- **MongoDB レプリケーション** - データベース冗長化
- **CDN** - 静的ファイル配信

## 🔒 セキュリティ対策

- **Helmet.js** - HTTP ヘッダーセキュリティ
- **CORS** - クロスオリジンリクエスト制御
- **JWT** - トークンベース認証
- **Passport.js** - 認証戦略管理
- **環境変数** - 機密情報管理
- **HTTPS** - 本番環境での暗号化通信

## 📊 パフォーマンス最適化

- **Redis キャッシング** - 頻繁にアクセスされるデータのキャッシュ
- **MongoDB インデックス** - クエリ最適化
- **ページネーション** - 大量データの処理
- **遅延ロード** - フロントエンドのパフォーマンス向上
- **画像最適化** - browser-image-compression

## 🧪 テスト戦略

### ユニットテスト
- コンポーネントのテスト
- ユーティリティ関数のテスト

### 統合テスト
- API エンドポイントのテスト
- データベース操作のテスト

### E2E テスト
- ユーザーフローのテスト
- ブラウザ操作のテスト

## 📚 ドキュメント

| ドキュメント | 説明 |
|-------------|------|
| README.md | プロジェクト概要 |
| QUICK_START.md | 5分で始めるガイド |
| SETUP_GUIDE.md | 詳細なセットアップ手順 |
| INTEGRATION_GUIDE.md | 新機能の統合ガイド |
| DATABASE_GUIDE.md | データベース管理ガイド |
| PROJECT_OVERVIEW.md | このファイル |

## 🚀 デプロイメント

### 開発環境
```bash
docker-compose up -d
```

### 本番環境
```bash
docker-compose -f docker-compose.prod.yml up -d
```

### クラウドデプロイ
- **AWS**: ECS, RDS, ElastiCache
- **GCP**: Cloud Run, Cloud SQL, Memorystore
- **Azure**: App Service, Cosmos DB, Azure Cache

## 📞 サポート

### よくある質問
- [QUICK_START.md](./QUICK_START.md) のトラブルシューティング
- [DATABASE_GUIDE.md](./DATABASE_GUIDE.md) のトラブルシューティング

### 問題報告
- GitHub Issues で報告
- ログを確認: `docker-compose logs -f`

## 🔄 開発ワークフロー

### 機能開発の流れ

1. **要件定義** - 機能の仕様を定義
2. **データモデル設計** - MongoDB スキーマを設計
3. **API 実装** - Express ルートを実装
4. **フロントエンド実装** - React コンポーネントを実装
5. **テスト** - ユニット・統合テストを実行
6. **デバッグ** - ログを確認して修正
7. **デプロイ** - 本番環境にデプロイ

## 📈 今後の拡張予定

### Phase 2
- Q&A・質問掲示板
- スケジュール・カレンダー統合
- 学習リソース共有

### Phase 3
- AI 学習アシスタント
- ビデオ通話・オンライン自習室
- 保護者向けダッシュボード

### Phase 4
- モバイルアプリ化
- 多言語対応
- アクセシビリティ改善

## 📄 ライセンス

ISC License

## 👥 チーム

- **開発者**: Ryo3-yo
- **プロジェクト**: Y-Hakua SNS

---

**最終更新**: 2026年2月1日
**バージョン**: 1.0.0
**ステータス**: 開発中

