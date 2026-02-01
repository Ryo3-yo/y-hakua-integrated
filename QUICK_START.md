# Y-Hakua SNS クイックスタートガイド

このガイドで、5分以内にY-Hakua SNS統合環境を起動できます。

## ⚡ 5分で始める

### ステップ1: プロジェクトディレクトリに移動

```bash
cd /home/ubuntu/y-hakua-integrated
```

### ステップ2: Docker Compose で起動

```bash
# すべてのサービスを起動
docker-compose up -d

# ログを確認（Ctrl+C で終了）
docker-compose logs -f
```

### ステップ3: ブラウザでアクセス

- **フロントエンド**: http://localhost:3000
- **バックエンド API**: http://localhost:8800
- **ヘルスチェック**: http://localhost:8800/health

### ステップ4: 初期データを挿入（オプション）

```bash
# テストデータを作成
docker-compose exec backend node scripts/seedData.js
```

## 📊 サービスの状態確認

```bash
# すべてのコンテナの状態を確認
docker-compose ps

# 出力例:
# NAME                COMMAND                  SERVICE             STATUS
# y-hakua-backend     "npm start"              backend             Up
# y-hakua-frontend    "npm start"              frontend            Up
# y-hakua-mongodb     "mongod"                 mongodb             Up
# y-hakua-redis       "redis-server"           redis               Up
```

## 🔍 ログの確認

```bash
# すべてのログを表示
docker-compose logs

# 特定のサービスのログを表示
docker-compose logs backend
docker-compose logs frontend

# リアルタイムでログを監視
docker-compose logs -f

# 最新の50行を表示
docker-compose logs --tail=50
```

## 🛑 サービスの停止

```bash
# すべてのサービスを停止
docker-compose stop

# 特定のサービスを停止
docker-compose stop backend

# すべてのコンテナを削除
docker-compose down

# ボリュームも削除
docker-compose down -v
```

## 🔄 サービスの再起動

```bash
# すべてのサービスを再起動
docker-compose restart

# 特定のサービスを再起動
docker-compose restart backend
```

## 🧪 API テスト

### ヘルスチェック

```bash
curl http://localhost:8800/health
```

### 学習記録を作成（トークンが必要）

```bash
# トークンを取得してから実行
curl -X POST http://localhost:8800/api/studylogs \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "数学",
    "durationMinutes": 60,
    "date": "2026-02-01",
    "notes": "テスト"
  }'
```

## 📱 フロントエンド機能

### 新機能へのアクセス

以下のコンポーネントが利用可能です：

1. **StudyLogForm** - 学習記録を追加
2. **StudyLogList** - 学習記録を表示
3. **GamificationDashboard** - ポイント・リーダーボード表示
4. **NoteEditor** - ノートを作成・編集
5. **StudyGroupCard** - 学習グループを表示

### App.js への統合例

```javascript
import { StudyLogProvider } from './state/StudyLogContext';
import { NoteProvider } from './state/NoteContext';
import { StudyGroupProvider } from './state/StudyGroupContext';
import { GamificationProvider } from './state/GamificationContext';

import StudyLogForm from './components/StudyLogForm';
import StudyLogList from './components/StudyLogList';
import GamificationDashboard from './components/GamificationDashboard';

function App() {
  return (
    <StudyLogProvider>
      <NoteProvider>
        <StudyGroupProvider>
          <GamificationProvider>
            <div>
              <StudyLogForm />
              <StudyLogList />
              <GamificationDashboard />
            </div>
          </GamificationProvider>
        </StudyGroupProvider>
      </NoteProvider>
    </StudyLogProvider>
  );
}
```

## 🗄️ データベース操作

### MongoDB に接続

```bash
# mongosh で接続
docker-compose exec mongodb mongosh -u root -p password

# y-hakua データベースに切り替え
use y-hakua

# コレクション一覧を表示
show collections

# ユーザーを確認
db.users.find().pretty()
```

### テストデータを作成

```bash
# 初期化スクリプトを実行
docker-compose exec backend node scripts/seedData.js
```

## 🐛 よくある問題

### ポート競合エラー

```
Error: listen EADDRINUSE: address already in use :::3000
```

**解決方法**: `docker-compose.yml` でポートを変更

```yaml
ports:
  - "3001:3000"  # 3000 → 3001
```

### MongoDB 接続エラー

```
MongooseError: Cannot connect to MongoDB
```

**解決方法**: MongoDB コンテナを再起動

```bash
docker-compose restart mongodb
docker-compose logs mongodb
```

### メモリ不足エラー

**解決方法**: Docker Desktop のメモリを増やす

- Settings > Resources > Memory を 4GB 以上に設定

## 📚 詳細ドキュメント

- [README.md](./README.md) - プロジェクト概要
- [SETUP_GUIDE.md](./SETUP_GUIDE.md) - 詳細セットアップ
- [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - 統合ガイド
- [DATABASE_GUIDE.md](./DATABASE_GUIDE.md) - データベース管理

## 🚀 次のステップ

1. **フロントエンド App.js を更新** - 新しいコンポーネントを追加
2. **ページを作成** - StudyLogsPage, NotesPage など
3. **ナビゲーション追加** - 新しいページへのリンク
4. **テスト** - ブラウザで動作確認
5. **カスタマイズ** - スタイルやロジックを調整

## 💡 ヒント

### ホットリロード

Docker で起動している場合、コード変更時に自動的にリロードされます。

```bash
# フロントエンドのコードを編集
code frontend/src/App.js

# 変更は自動的に反映されます
```

### ログの監視

開発中はログを監視することをお勧めします。

```bash
# 別のターミナルでログを監視
docker-compose logs -f
```

### コンテナの再構築

依存関係を変更した場合は、イメージを再ビルドしてください。

```bash
docker-compose build --no-cache
docker-compose up -d
```

## 📧 サポート

問題が発生した場合:

1. ログを確認: `docker-compose logs -f`
2. このガイドのトラブルシューティングを参照
3. 詳細ドキュメントを確認
4. GitHub Issues で報告

---

**準備完了！楽しい開発を！** 🎉

