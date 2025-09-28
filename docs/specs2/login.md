# ログイン画面機能仕様書

<img src="https://hackmd.io/_uploads/S1mq_IIhge.png" width=620px>

## 1. 概要
- 会員制投資コミュニティ「すなっちゃん」へアクセスする入口となる認証画面。
- Firebase Auth（Googleプロバイダ）を利用したOAuthログインを採用する。
- 成功時は投資ダッシュボードへ遷移し、初回ログイン時はオンボーディングモーダルを表示予定。

## 2. UI/UXデザイン
- シングルカラム構成（モバイル優先）、デスクトップではヒーローセクション＋補助情報の2カラム展開。
- ブランドパレット：メイン #FF6B35、アクセント #1B1F3B、ニュートラルグレー。角丸4px・多層ドロップシャドウを共通適用。
- フォント：Google Sans（英数）、Noto Sans JP（日本語）。CTAはフィルボタン＋十分な余白で視認性確保。
- コンテンツ要素：コミュニティロゴ、ヒーローコピー、CTAボタン2種（Googleログイン、Stripeログイン）
- レスポンシブ方針：600px未満は縦積み、600-1024pxは中央寄せ、1024px以上は左右レイアウト。背景に軽いグラデーションやブランドイラストを使用。

## 3. 画面遷移フロー
- 既存セッショントークンが有効？
    - 有効
        - 前回のバージョンより更新されている
            1. バージョンオンボーディングモーダルを表示
        1. 投資ダッシュボードへ自動リダイレクト
    - セッションなし
        1. ログイン画面に滞在
        1. 「Googleでログイン」クリック
        1. GoogleのOAuth認可画面へ遷移。
        - 認可成功
            1. サーバー側でセッション生成
            1. 投資ダッシュボードへ遷移
            - 初回ログインの場合
                1. オンボーディングモーダルを表示
                1. 完了後ダッシュボード利用開始
- ログアウト実行（任意）
    1. セッション無効化
    2. 再び /login へ戻る


## 4. ER図
```plantuml
@startuml
@startuml
  hide circle
  skinparam linetype ortho

  entity "User" as User {
    *id: bigint
    --
    firebase_uuid: string
    email: string
    name: string
    role: string
    stripe_customer_id: string?
    onboarding_completed_at: datetime?
    created_at: datetime
    updated_at: datetime
  }

  entity "Subscription" as Subscription {
    *id: bigint
    --
    user_id: bigint
    stripe_subscription_id: string
    plan: string
    status: string
    trial_end_at: datetime?
    created_at: datetime
    updated_at: datetime
  }

  entity "Session" as Session {
    *id: bigint
    --
    user_id: bigint
    access_token: string
    refresh_token: string
    user_agent: string?
    expires_at: datetime
    created_at: datetime
  }

  User ||--|| Subscription : "1 : 1"
  User ||--o{ Session : "1 : N"

  @enduml
```
- User.role: `Owner` / `Member`
- User.login_uuid: ログインしているサービスの識別子
- Subscription.status: `trialing` / `active` / `past_due` / `canceled` を想定
- Session: Token管理用（要採用スタックに合わせて調整）

## 5. エンドポイント(TBD)
- GET `/login`：サーバーサイドレンダリングで画面描画。セッション有効時は302で `/` へ。
- POST `/api/auth/firebase/login`：Firebase AuthのIDトークン受領→検証→セッションクッキー発行。
- POST `/api/auth/firebase/logout`：Firebaseセッションクッキー失効、Refreshトークン失効。
- GET `/api/auth/firebase/session`：セッション検証用エンドポイント（フロント初期化時に利用）。
- Stripe Customer PortalやWebhook連携は本機能と同時構築予定。

## 6. 機能構成（階層構造）(TBD)
- `pages/login`（または`app/login/page`）：ページコンテナ。サーバーサイドレンダリングでセッション判定、UIレンダリング。
  - `components/LoginHero`：ヒーローコピー・補足説明。
  - `components/PricingCard`：料金・トライアル情報の表示。
  - `components/GoogleSignInButton`：OAuth開始処理、ローディング状態表示。
  - `components/TrialCtaButton`：トライアル詳細モーダルやFAQへの導線。
- `modules/auth`：Firebase Authとサーバーセッションの橋渡し、ユーザー同期。
- `modules/onboarding`：初回フラグ判定とモーダル表示制御。

## 7. データ管理方針(TBD)
- Firebase Authのクレデンシャル（APIキー、Service Account）は環境変数で管理。フロント側は公開APIキーのみ参照。
- セッションはFirebaseセッションクッキー＋短期アクセストークンと長期リフレッシュトークンの二段構成。
- Stripeサブスクリプション情報はWebhook連携を本機能と同時構築し、User/Subscriptionを更新。ログイン時に最新状態を参照。
- トライアル終了日時やプラン情報はCMSまたは設定テーブルで集中管理し、画面表示と整合性を確保。
- オンボーディング完了フラグはUserメタデータに保存し、サーバー判定を優先。

## 8. 実装上の注意点(TBD)
- OAuthリクエストに`state`と`nonce`を付与しCSRF・リプレイ攻撃を防ぐ（Firebase Authの`auth_nonce`利用）。
- サーバーサイドレンダリングでセッション判定することでFlicker（認証済→再リダイレクト）を防止。
- トライアル表示値はキャッシュ使用時でも有効期限を設定し、変更時に即時反映できるようにする。
- Stripe Customer Portalへのリンクはログイン後メニューに限定。未課金ユーザーは利用できないUIとする。
- 初回オンボーディングはフロントのローカルストレージとサーバーフラグを併用し、誤再表示を抑制。
- オンボーディングモーダルの内容は現時点で固定要件なし。将来的な文言差し替えに備え、設定値またはCMSから取得できる設計とする。

## 9. エラーハンドリング(TBD)
- OAuth失敗：トースト表示（例「Googleログインに失敗しました」）＋再試行ボタン。ログに詳細エラーIDを残す。
- サブスクリプション失効/未登録：ダッシュボード遷移前に課金再開案内ダイアログを表示。Portalへ遷移するCTAを用意。
- ネットワーク障害：指数バックオフで再試行。一定回数失敗でサポート窓口を案内。

## 10. 今後の拡張性
- 追加認証手段の導入（Email/Password、Apple Sign In、Slack SSOなど）。
- オンボーディングにアンケートやチュートリアルを組み込み、ユーザープロファイル収集を強化。
- ログイン画面に最新アップデート、動画紹介、ユーザーボイスなどマーケティング要素を追加。
- ABテストでCTA文言・料金表示レイアウトを最適化しコンバージョン向上。

---
## 要確認事項
- 現時点なし
