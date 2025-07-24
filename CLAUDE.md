# SnatchTradingChatApp - CLAUDE開発ガイド

## プロジェクト概要

SnatchTradingChatAppは投資コミュニティ向けのチャットアプリケーションです。X（Twitter）認証、リアルタイムチャット、投資記事管理、ボット連携、Stripe決済などの機能を提供します。

## アーキテクチャ

### 技術スタック
- **バックエンド**: Node.js + Express.js
- **データベース**: PostgreSQL + Prisma ORM
- **キャッシュ**: Redis
- **認証**: X（Twitter） OAuth2.0 + JWT
- **決済**: Stripe
- **リアルタイム通信**: WebSocket (Socket.io)
- **ファイルストレージ**: AWS S3 / Google Cloud Storage
- **フロントエンド**: React + Redux

### データベース設計
PostgreSQLを使用し、以下の主要テーブルで構成：
- users（ユーザー管理）
- channels（チャンネル管理）
- messages（メッセージ）
- articles（記事）
- subscriptions（サブスク）
- notifications（通知）
- bots（ボット）

## 機能一覧

### 高優先度機能

#### 1. 認証・アカウント管理 (`docs/specs/auth_account_spec.md`)
- X（Twitter）OAuth認証によるログイン
- ユーザープロフィール管理（表示名、自己紹介、アバター）
- ロールベースアクセス制御（管理者、一般ユーザー、ゲスト）
- セッション管理とトークンリフレッシュ

**主要エンドポイント**:
- `POST /api/auth/login/twitter` - X認証開始
- `GET /api/auth/callback/twitter` - OAuth コールバック
- `GET /api/users/me` - プロフィール取得
- `PUT /api/users/me` - プロフィール更新

#### 2. チャット機能 (`docs/specs/chat_spec.md`)
- リアルタイムメッセージ送受信
- リッチテキスト対応（太字、斜体、コードブロック等）
- ファイル添付（画像、動画、ドキュメント）
- 絵文字リアクション
- スレッド形式の返信
- ダイレクトメッセージ（1対1）

**主要エンドポイント**:
- `GET /api/channels/{channelId}/messages` - メッセージ取得
- `POST /api/channels/{channelId}/messages` - メッセージ送信
- `POST /api/messages/{messageId}/reactions` - リアクション追加
- `WS /ws/chat` - WebSocket接続

#### 3. チャンネル管理 (`docs/specs/channel_management_spec.md`)
- パブリック/プライベートチャンネル
- チャンネル作成・編集・削除
- メンバー招待・管理
- 権限設定（閲覧のみ、投稿権限など）

**主要エンドポイント**:
- `GET /api/channels` - チャンネル一覧
- `POST /api/channels` - チャンネル作成
- `POST /api/channels/{channelId}/members` - メンバー追加

#### 4. 通知システム (`docs/specs/notification_system_spec.md`)
- プッシュ通知（iOS/Android/Web）
- アプリ内通知
- メール通知
- 通知設定のカスタマイズ（カテゴリー別、時間帯設定）

**主要エンドポイント**:
- `GET /api/notifications` - 通知一覧取得
- `PUT /api/notification-settings` - 通知設定更新
- `POST /api/push-tokens` - プッシュトークン登録

#### 5. 記事管理 (`docs/specs/article_management_spec.md`)
- 投資記事の作成・編集・公開（管理者のみ）
- リッチテキストエディタ
- 画像・グラフの埋め込み
- いいね・コメント機能
- カテゴリー・タグ管理

**主要エンドポイント**:
- `GET /api/articles` - 記事一覧取得
- `POST /api/articles` - 記事作成（管理者のみ）
- `POST /api/articles/{articleId}/like` - いいね

#### 6. ボット機能 (`docs/specs/bot_integration_spec.md`)
- Cronによる定期実行
- 投資情報の自動取得・投稿
- カスタムスクリプト実行
- Webhook連携

**主要エンドポイント**:
- `GET /api/bots` - ボット一覧取得
- `POST /api/bots` - ボット作成
- `POST /api/bots/{botId}/execute` - 手動実行

#### 7. 決済・サブスクリプション (`docs/specs/payment_subscription_spec.md`)
- Stripe連携
- 月額/年額サブスクリプション
- プラン変更・キャンセル
- 請求履歴管理

**主要エンドポイント**:
- `GET /api/plans` - プラン一覧
- `POST /api/subscription` - サブスク作成
- `GET /api/invoices` - 請求履歴

#### 8. 管理者機能 (`docs/specs/admin_features_spec.md`)
- ユーザー管理・権限変更
- 入退会ログのCSVエクスポート
- システム設定管理
- レポート生成

**主要エンドポイント**:
- `GET /api/admin/users` - ユーザー一覧
- `PUT /api/admin/users/{userId}/roles` - 権限変更
- `GET /api/admin/reports` - レポート一覧

#### 9. 設定画面 (`docs/specs/settings_screen_spec.md`)
- アカウント設定（プロフィール、X連携）
- 通知設定（詳細カスタマイズ）
- セキュリティ設定
- サブスクリプション管理

### 中優先度機能
- メッセージ・ユーザー・チャンネルの検索機能

### 低優先度機能
- CSV出力による管理者機能（入退室者ログ）

## 開発における重要なポイント

### セキュリティ
- OAuth2.0による安全な認証
- JWTトークンの適切な管理
- CSRFおよびXSS対策
- 入力値のサニタイズ
- API呼び出しの権限チェック

### パフォーマンス
- メッセージの効率的なページネーション
- Redis によるキャッシング戦略
- WebSocket接続の最適化
- 画像・動画の遅延読み込み

### リアルタイム通信
- WebSocket (Socket.io) によるリアルタイムメッセージ
- 接続状態の管理
- メッセージの確実な配信
- オフライン時の対応

### データ管理
- PostgreSQL での構造化データ
- Redis でのキャッシュとセッション管理
- S3/Cloud Storage でのファイル管理
- 適切なインデックス設計

## 外部サービス連携

### X（Twitter）API
- OAuth2.0認証フロー
- ユーザー情報取得
- 認証トークンの管理

### Stripe
- サブスクリプション管理
- 決済処理
- Webhook による状態同期
- PCI DSS準拠

### WebSocket
- リアルタイムメッセージ配信
- プレゼンス管理
- スケーラブルな接続管理

## エラーハンドリング

### 一般的なHTTPステータスコード
- `401 Unauthorized`: 未認証
- `403 Forbidden`: 権限不足
- `404 Not Found`: リソースが存在しない
- `409 Conflict`: データの競合
- `429 Too Many Requests`: レート制限

### リアルタイム通信エラー
- WebSocket接続エラー時の自動再接続
- メッセージ送信失敗時のリトライ機能
- オフライン時のローカルキュー

## 開発・デプロイ環境

### 非機能要件
- レスポンスタイム: 平均2秒以内
- 同時接続ユーザー数: 1000人以上対応
- 稼働率: 99.9%以上
- SSL/TLS暗号化通信

### 推奨開発フロー
1. 要件定義書・仕様書の詳細確認
2. データベース設計の実装
3. 認証システムの構築
4. 基本的なCRUD API の実装
5. WebSocket による リアルタイム機能
6. フロントエンド との連携
7. 外部サービス（Stripe、X API）との統合
8. テスト・デバッグ
9. セキュリティチェック
10. デプロイ・運用

## トラブルシューティング

### よくある問題
1. **WebSocket接続が切れる**: 自動再接続機能の実装を確認
2. **認証エラー**: JWTトークンの有効期限とリフレッシュ処理を確認
3. **Stripe Webhook失敗**: 署名検証とエラーハンドリングを確認
4. **パフォーマンス低下**: データベースインデックスとキャッシュ戦略を見直し

### ログ確認箇所
- アプリケーションログ（エラー、API呼び出し）
- WebSocketログ（接続・切断、メッセージ配信）
- データベースログ（スロークエリ）
- 外部API連携ログ（X API、Stripe）

## ディレクトリ構造

```
/
├── docs/                    # プロジェクトドキュメント
│   ├── requirements.md      # 要件定義書
│   └── specs/              # 各機能の詳細仕様
├── src/                    # アプリケーション ソースコード（想定）
├── tests/                  # テストファイル（想定）
├── config/                 # 設定ファイル（想定）
└── CLAUDE.md              # このファイル
```

## 追加の注意事項

- 新機能開発時は該当する仕様書（`docs/specs/`）を必ず参照
- データベース変更時はマイグレーションファイルを作成
- API変更時は適切なバージョニングを実施
- セキュリティ関連の変更は必ずレビューを実施
- パフォーマンスに影響する変更は事前に負荷テストを実施

このドキュメントは開発進捗に合わせて随時更新してください。
