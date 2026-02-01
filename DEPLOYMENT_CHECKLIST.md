# Y-Hakua SNS デプロイメントチェックリスト

本番環境へのデプロイメント前に、このチェックリストを確認してください。

## ✅ 開発環境チェック

- [ ] Docker & Docker Compose がインストールされている
- [ ] Node.js 20+ がインストールされている
- [ ] Git がインストールされている
- [ ] すべての依存パッケージがインストールされている
  ```bash
  cd frontend && npm install
  cd ../backend && npm install
  ```

## ✅ 環境変数チェック

### バックエンド (.env)
- [ ] `MONGO_URI` が設定されている
- [ ] `REDIS_URL` が設定されている
- [ ] `JWT_SECRET` が強力なものに変更されている
- [ ] `GOOGLE_CLIENT_ID` が設定されている
- [ ] `GOOGLE_CLIENT_SECRET` が設定されている
- [ ] `CLOUDINARY_NAME` が設定されている（オプション）
- [ ] `CLOUDINARY_API_KEY` が設定されている（オプション）
- [ ] `CLOUDINARY_API_SECRET` が設定されている（オプション）
- [ ] `NODE_ENV` が `production` に設定されている

### フロントエンド (.env.production)
- [ ] `REACT_APP_API_URL` が本番 API URL に設定されている
- [ ] `REACT_APP_GOOGLE_CLIENT_ID` が設定されている

## ✅ データベースチェック

- [ ] MongoDB が正常に起動している
- [ ] 初期データが挿入されている
  ```bash
  docker-compose exec backend node scripts/seedData.js
  ```
- [ ] バックアップが取得されている
- [ ] インデックスが作成されている
- [ ] ユーザーアカウントが作成されている

## ✅ セキュリティチェック

- [ ] JWT_SECRET が変更されている
- [ ] MongoDB ユーザーのパスワードが変更されている
- [ ] Redis がパスワード保護されている
- [ ] CORS が正しく設定されている
- [ ] Helmet.js が有効になっている
- [ ] HTTPS が有効になっている（本番環境）
- [ ] 環境変数が `.gitignore` に含まれている
- [ ] API キーが環境変数で管理されている

## ✅ パフォーマンスチェック

- [ ] フロントエンドがビルドされている
  ```bash
  cd frontend && npm run build
  ```
- [ ] 不要なコンソールログが削除されている
- [ ] キャッシング戦略が実装されている
- [ ] 画像が最適化されている
- [ ] バンドルサイズが確認されている
- [ ] ページロード時間が測定されている

## ✅ テストチェック

- [ ] ユニットテストが実行されている
- [ ] 統合テストが実行されている
- [ ] E2E テストが実行されている
- [ ] エラーハンドリングがテストされている
- [ ] エッジケースがテストされている

## ✅ API チェック

- [ ] すべてのエンドポイントが動作している
- [ ] エラーレスポンスが正しい形式である
- [ ] 認証が正しく機能している
- [ ] レート制限が実装されている
- [ ] ログが記録されている

## ✅ フロントエンドチェック

- [ ] すべてのページが表示されている
- [ ] フォームが正しく動作している
- [ ] バリデーションが機能している
- [ ] エラーメッセージが表示されている
- [ ] レスポンシブデザインが確認されている
- [ ] ブラウザ互換性が確認されている
- [ ] アクセシビリティが確認されている

## ✅ Docker チェック

- [ ] Dockerfile が最適化されている
- [ ] イメージサイズが確認されている
- [ ] マルチステージビルドが使用されている（オプション）
- [ ] ヘルスチェックが設定されている
- [ ] ボリュームマウントが正しく設定されている
- [ ] ネットワークが正しく設定されている

## ✅ ドキュメントチェック

- [ ] README.md が最新である
- [ ] API ドキュメントが完成している
- [ ] セットアップガイドが完成している
- [ ] トラブルシューティングガイドが完成している
- [ ] 変更ログが記録されている

## ✅ モニタリングチェック

- [ ] ログ収集が設定されている
- [ ] エラートラッキングが設定されている（Sentry など）
- [ ] パフォーマンスモニタリングが設定されている
- [ ] アラートが設定されている
- [ ] ダッシュボードが作成されている

## ✅ バックアップチェック

- [ ] データベースバックアップが設定されている
- [ ] バックアップの復元手順が確認されている
- [ ] 定期的なバックアップスケジュールが設定されている
- [ ] バックアップストレージが確保されている

## ✅ スケーリングチェック

- [ ] 負荷テストが実行されている
- [ ] スケーリング戦略が定義されている
- [ ] キャッシング戦略が実装されている
- [ ] データベース接続プーリングが設定されている
- [ ] CDN が設定されている（オプション）

## ✅ 本番環境セットアップ

### AWS への デプロイ例

```bash
# ECR にイメージをプッシュ
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

docker tag y-hakua-backend:1.0.0 <account-id>.dkr.ecr.us-east-1.amazonaws.com/y-hakua-backend:1.0.0
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/y-hakua-backend:1.0.0

# ECS タスク定義を更新
aws ecs update-service --cluster y-hakua --service backend --force-new-deployment

# RDS インスタンスを作成
aws rds create-db-instance \
  --db-instance-identifier y-hakua-mongodb \
  --db-instance-class db.t3.micro \
  --engine mongodb

# ElastiCache インスタンスを作成
aws elasticache create-cache-cluster \
  --cache-cluster-id y-hakua-redis \
  --cache-node-type cache.t3.micro \
  --engine redis
```

### GCP への デプロイ例

```bash
# Cloud Run にデプロイ
gcloud run deploy y-hakua-backend \
  --source . \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated

# Cloud SQL インスタンスを作成
gcloud sql instances create y-hakua-mongodb \
  --database-version MONGODB_7_0 \
  --tier db-f1-micro

# Memorystore インスタンスを作成
gcloud redis instances create y-hakua-redis \
  --size=1 \
  --region=us-central1
```

## ✅ デプロイ後チェック

- [ ] アプリケーションが起動している
- [ ] ヘルスチェックエンドポイントが応答している
- [ ] ログが正常に記録されている
- [ ] データベースに接続できている
- [ ] キャッシュが機能している
- [ ] メール通知が送信されている
- [ ] 外部 API が接続できている
- [ ] ユーザーが登録できている
- [ ] ユーザーがログインできている
- [ ] 新機能が動作している

## 🔄 ロールバック手順

問題が発生した場合の対応:

```bash
# 前のバージョンに戻す
docker-compose down
git checkout <previous-commit>
docker-compose build --no-cache
docker-compose up -d

# または、イメージを指定
docker-compose -f docker-compose.yml up -d --image y-hakua-backend:1.0.0-previous
```

## 📞 サポート連絡先

- **開発チーム**: dev-team@example.com
- **運用チーム**: ops-team@example.com
- **セキュリティ**: security@example.com

## 📋 デプロイ記録

| 日時 | バージョン | 環境 | ステータス | 備考 |
|------|-----------|------|-----------|------|
| 2026-02-01 | 1.0.0 | 開発 | ✅ 完了 | 初回デプロイ |
| | | 本番 | ⏳ 予定 | |

## ✅ 最終確認

- [ ] すべてのチェックリスト項目が完了している
- [ ] 管理者の承認が得られている
- [ ] デプロイ予定日時が確認されている
- [ ] ロールバック計画が準備されている
- [ ] ユーザーへの通知が準備されている
- [ ] 緊急対応チームが待機している

---

**デプロイ日時**: ________________
**デプロイ担当者**: ________________
**承認者**: ________________

**最終更新**: 2026年2月1日

