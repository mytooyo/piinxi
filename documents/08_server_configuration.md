# 📁 piinxi Rustサーバー構成（CQRS + BFF + sqlx）

## 🔹 ルート構成例

```text
piinxi-backend/
├── Cargo.toml
├── .env
├── schema.sql           # DBマイグレーション定義（sqlx対応）
├── src/
│   ├── main.rs
│   ├── lib.rs
│   ├── routes/          # axumルーティング（Handler定義）
│   │   ├── mod.rs
│   │   ├── users.rs
│   │   ├── nests.rs
│   │   └── piins.rs
│   ├── domain/          # ドメインモデル（DB構造に近い）
│   │   ├── mod.rs
│   │   ├── models/
│   │   │   ├── user.rs
│   │   │   ├── nest.rs
│   │   │   └── piin.rs
│   ├── application/     # CQRSのユースケース（DTO含む）
│   │   ├── mod.rs
│   │   ├── queries/
│   │   │   ├── get_user.rs
│   │   │   ├── list_nests.rs
│   │   │   └── latest_piin.rs
│   │   ├── commands/
│   │   │   ├── create_user.rs
│   │   │   ├── update_user.rs
│   │   │   └── delete_user.rs
│   │   ├── dtos/
│   │   │   ├── user_dto.rs
│   │   │   ├── nest_dto.rs
│   │   │   └── piin_dto.rs
│   ├── infrastructure/  # 外部接続（DB, FCM）
│   │   ├── mod.rs
│   │   ├── db.rs        # PgPool管理
│   │   └── repositories/
│   │       ├── user_repository.rs
│   │       ├── nest_repository.rs
│   │       └── piin_repository.rs
│   ├── config/           # .env読み込み & アプリ設定
│   │   └── settings.rs
```

---

## 🧠 補足ポイント

| フォルダ | 内容 |
|---------|------|
| `routes/` | axumのHandler、`GET /users/me`など |
| `domain/models/` | 生のDBエンティティ（sqlx::FromRow） |
| `application/commands/` | 書き込み操作（POST/PUT/DELETE） |
| `application/queries/` | 読み取り専用（GET系） |
| `application/dtos/` | フロントエンド向けの整形済レスポンス型 |
| `infrastructure/repositories/` | DB操作ロジック（生SQL） |

---

## ✅ 特徴的な設計意図

- **CQRS分離**：読み取りと更新をしっかり分けて管理・保守しやすく
- **DTO化**：クライアントに最適化された構造で返却しやすい
- **クリーンな責務分離**：infra, domain, application, route それぞれ明確
