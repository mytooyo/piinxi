# 🧰 piinxi 技術スタック構成（Flutter × Rust）

## 📱 クライアント（Flutter）

| 項目           | 使用技術・ライブラリ                          |
|----------------|-----------------------------------------------|
| UIフレームワーク | Flutter（Material3 + アニメーション）          |
| 状態管理       | `riverpod` or `flutter_bloc`（アーキテクチャ向き） |
| 通信           | `dio` or `http`（REST API呼び出し）            |
| 通知           | `firebase_messaging`（プッシュ通知）           |
| アニメーション | `rive` or `lottie`（キャラや演出表現）         |
| QR招待         | `qr_flutter` or `mobile_scanner`              |

---

## 🖥️ サーバーサイド（Rust）

| 項目             | 使用技術・ライブラリ                                 |
|------------------|------------------------------------------------------|
| Webフレームワーク | `axum`（非同期対応、構造化しやすい）                 |
| データベース     | `PostgreSQL` + `sqlx` or `SeaORM`                    |
| 認証管理         | `jsonwebtoken`, `argon2`（匿名ニックネームで対応）  |
| Nest管理         | `uuid`でNestコード発行、ユーザーとの紐付け           |
| 通知連携         | `Firebase Cloud Messaging`（FCM API送信）           |
| インフラ構成     | AWS Lambda + API Gateway or Cloud Run               |
| デプロイ         | `cargo-lambda` + `Terraform` or `CDK`               |

---

## 🔄 通信方式

| 通信方式          | 用途                                |
|-------------------|-------------------------------------|
| REST API          | piin送信、nest作成、ユーザー登録など |
| WebSocket（将来的） | nest内の演出同期など                |
| Push通知（FCM）    | piinの受信通知（静かに伝えるため）  |

---

## 🧪 ローカル開発環境

- PostgreSQL：`Docker Compose`
- FCM：Firebase Emulator Suite（または本番環境で限定公開）
- Rust開発：`cargo watch` + `tokio-console`（開発効率化）
- Flutter：`Flutter devtools` + `iOS/Android emulator`

---

## ✨ この構成の強み

- Flutterで **優しいUI/UXとアニメーション表現**
- Rustで **堅牢かつ軽量なバックエンド**
- Firebaseで **通知だけ活用してスリムな実装**

---

## ✅ 次に決めること候補

- [ ] API設計（Rust側のエンドポイントと責務設計）
- [ ] nest / user / piin のDBスキーマ設計
- [ ] クライアント↔サーバ間の通信インターフェース設計
