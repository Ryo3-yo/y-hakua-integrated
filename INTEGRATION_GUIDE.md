# Y-Hakua SNS 統合ガイド

このドキュメントは、新機能をY-Hakua SNSアプリケーションに統合するための詳細なガイドです。

## 📦 統合済みコンポーネント

### バックエンド統合

#### 1. 新しいモデル

```
backend/models/
├── StudyLog.js                 # 学習記録
├── Note.js                     # ノート
├── UserAssignmentStatus.js     # 課題ステータス
├── StudyGroup.js               # 学習グループ
└── UserAchievement.js          # ゲーミフィケーション
```

#### 2. 新しいAPI ルート

```
backend/routes/
├── studylogs.js                # 学習記録API
├── notes.js                    # ノートAPI
├── studygroups.js              # 学習グループAPI
└── gamification.js             # ゲーミフィケーションAPI
```

#### 3. server.js への統合

```javascript
// 新機能ルートをマウント
app.use('/api/studylogs', require('./routes/studylogs'));
app.use('/api/notes', require('./routes/notes'));
app.use('/api/studygroups', require('./routes/studygroups'));
app.use('/api/gamification', require('./routes/gamification'));
```

### フロントエンド統合

#### 1. 状態管理コンテキスト

```
frontend/src/state/
├── StudyLogContext.js          # 学習記録の状態管理
├── NoteContext.js              # ノートの状態管理
├── StudyGroupContext.js        # 学習グループの状態管理
└── GamificationContext.js      # ゲーミフィケーションの状態管理
```

#### 2. コンポーネント

```
frontend/src/components/
├── StudyLogForm.js             # 学習記録フォーム
├── StudyLogList.js             # 学習記録一覧
├── GamificationDashboard.js    # ゲーミフィケーション表示
├── NoteEditor.js               # ノートエディタ
└── StudyGroupCard.js           # 学習グループカード
```

## 🔧 フロントエンド App.js への統合

以下のコンテキストプロバイダーを App.js に追加してください：

```javascript
import { StudyLogProvider } from './state/StudyLogContext';
import { NoteProvider } from './state/NoteContext';
import { StudyGroupProvider } from './state/StudyGroupContext';
import { GamificationProvider } from './state/GamificationContext';

function App() {
  return (
    <StudyLogProvider>
      <NoteProvider>
        <StudyGroupProvider>
          <GamificationProvider>
            {/* アプリケーションのコンテンツ */}
          </GamificationProvider>
        </StudyGroupProvider>
      </NoteProvider>
    </StudyLogProvider>
  );
}
```

## 📄 ページの作成例

### 学習記録ページ

```javascript
// frontend/src/pages/StudyLogsPage.js
import React, { useState } from 'react';
import { Container, Box } from '@mui/material';
import StudyLogForm from '../components/StudyLogForm';
import StudyLogList from '../components/StudyLogList';

const StudyLogsPage = () => {
  const [filters, setFilters] = useState({});

  return (
    <Container maxWidth="lg" sx={{ py: 4 }}>
      <StudyLogForm onSuccess={() => {}} />
      <StudyLogList filters={filters} />
    </Container>
  );
};

export default StudyLogsPage;
```

### ゲーミフィケーションページ

```javascript
// frontend/src/pages/GamificationPage.js
import React from 'react';
import { Container } from '@mui/material';
import GamificationDashboard from '../components/GamificationDashboard';

const GamificationPage = () => {
  return (
    <Container maxWidth="lg" sx={{ py: 4 }}>
      <GamificationDashboard />
    </Container>
  );
};

export default GamificationPage;
```

## 🔌 API 使用例

### 学習記録を作成

```javascript
const response = await fetch('/api/studylogs', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({
    subject: '数学',
    durationMinutes: 60,
    date: '2026-02-01',
    notes: '微分積分の復習',
  }),
});
```

### ノートを作成

```javascript
const response = await fetch('/api/notes', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({
    title: 'Python学習ノート',
    content: '# Pythonの基本\n\n...',
    isPublic: true,
    tags: ['Python', '初心者'],
  }),
});
```

### 学習グループに参加

```javascript
const response = await fetch('/api/studygroups/join/groupId', {
  method: 'PUT',
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

## 🧪 テスト

### バックエンド API テスト

```bash
# 学習記録の作成テスト
curl -X POST http://localhost:8800/api/studylogs \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "数学",
    "durationMinutes": 60,
    "date": "2026-02-01",
    "notes": "テスト"
  }'

# 学習記録の取得テスト
curl -X GET http://localhost:8800/api/studylogs \
  -H "Authorization: Bearer YOUR_TOKEN"

# リーダーボード取得テスト
curl -X GET http://localhost:8800/api/gamification/leaderboard \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### フロントエンド テスト

```javascript
// useStudyLogs フック のテスト
import { useStudyLogs } from '../state/StudyLogContext';

const TestComponent = () => {
  const { logs, createLog, loading } = useStudyLogs();

  const handleTest = async () => {
    await createLog({
      subject: 'テスト',
      durationMinutes: 30,
      date: new Date().toISOString().split('T')[0],
    });
  };

  return <button onClick={handleTest}>テスト</button>;
};
```

## 🔐 認証とセキュリティ

### JWT トークンの管理

すべてのAPIリクエストに JWT トークンを含める必要があります：

```javascript
const token = localStorage.getItem('token');
const headers = {
  'Authorization': `Bearer ${token}`,
  'Content-Type': 'application/json',
};
```

### トークンの更新

トークンが期限切れの場合は、リフレッシュトークンを使用して新しいトークンを取得してください：

```javascript
const refreshToken = async () => {
  const response = await fetch('/api/auth/refresh', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      refreshToken: localStorage.getItem('refreshToken'),
    }),
  });
  const data = await response.json();
  localStorage.setItem('token', data.token);
};
```

## 📊 データベース初期化

### MongoDB への初期データ挿入

```javascript
// backend/scripts/seedData.js
const mongoose = require('mongoose');
const User = require('../models/User');
const StudyGroup = require('../models/StudyGroup');

const seedData = async () => {
  await mongoose.connect(process.env.MONGO_URI);

  // テストユーザーを作成
  const user = await User.create({
    name: 'テストユーザー',
    email: 'test@example.com',
    password: 'hashed_password',
  });

  // テストグループを作成
  await StudyGroup.create({
    name: 'Python学習グループ',
    description: 'Pythonの勉強をしている人のグループ',
    adminId: user._id,
    members: [user._id],
    topics: ['Python', 'プログラミング'],
  });

  console.log('初期データを挿入しました');
  process.exit(0);
};

seedData().catch(err => {
  console.error(err);
  process.exit(1);
});
```

実行方法：

```bash
docker-compose exec backend node scripts/seedData.js
```

## 🚀 本番環境への展開

### 環境変数の設定

```bash
# .env.production を作成
NODE_ENV=production
MONGO_URI=mongodb+srv://user:password@cluster.mongodb.net/y-hakua
REDIS_URL=redis://redis-host:6379
JWT_SECRET=your_strong_secret_key
FRONTEND_URL=https://yourdomain.com
```

### Docker イメージのビルド

```bash
# バックエンド
docker build -t y-hakua-backend:1.0.0 ./backend

# フロントエンド
docker build -t y-hakua-frontend:1.0.0 ./frontend
```

### デプロイメント

```bash
# Kubernetes を使用する場合
kubectl apply -f k8s/deployment.yaml

# または Docker Compose を使用
docker-compose -f docker-compose.prod.yml up -d
```

## 🐛 トラブルシューティング

### API エラー 401 Unauthorized

**原因**: トークンが無効または期限切れ

**解決方法**:
```javascript
// トークンを再取得
const response = await fetch('/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password }),
});
const data = await response.json();
localStorage.setItem('token', data.token);
```

### API エラー 500 Server Error

**原因**: サーバーエラー

**解決方法**:
```bash
# バックエンドのログを確認
docker-compose logs -f backend

# エラーメッセージを確認して修正
```

### コンポーネントが描画されない

**原因**: Context Provider が設定されていない

**解決方法**:
```javascript
// App.js で全てのプロバイダーをラップ
<StudyLogProvider>
  <NoteProvider>
    <StudyGroupProvider>
      <GamificationProvider>
        {/* コンテンツ */}
      </GamificationProvider>
    </StudyGroupProvider>
  </NoteProvider>
</StudyLogProvider>
```

## 📚 参考資料

- [実装ガイド](./implementation_guide.md)
- [セットアップガイド](./SETUP_GUIDE.md)
- [README](./README.md)
- [React Context API ドキュメント](https://react.dev/reference/react/useContext)
- [Express ドキュメント](https://expressjs.com)
- [MongoDB ドキュメント](https://docs.mongodb.com)

## 🤝 サポート

問題が発生した場合は、以下の手順で対応してください：

1. ログを確認: `docker-compose logs -f`
2. このガイドのトラブルシューティングセクションを参照
3. GitHub Issues で報告

---

**最終更新**: 2026年2月1日
**バージョン**: 1.0.0
