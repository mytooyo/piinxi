# 🗂️ piinxi DBスキーマ設計（MVP構成）

## ✅ コンセプト

- 履歴を持ちすぎず、「今この瞬間」のやりとりに集中
- ユーザー情報は最小限。匿名 or ニックネームだけでOK
- nestごとの関係性と送信ログがあれば成立する

---

## 🧑 users テーブル

| カラム名      | 型        | 説明                           |
|---------------|-----------|--------------------------------|
| id            | UUID      | 主キー                         |
| nickname      | TEXT      | 表示用ニックネーム（匿名可）   |
| avatar_url    | TEXT      | プロフィール画像URL（任意）    |
| fcm_token     | TEXT      | FCM送信用トークン              |
| created_at    | TIMESTAMP | 登録日時                       |
| updated_at    | TIMESTAMP | 最終更新日時                   |

---

## 🪺 nests テーブル

| カラム名      | 型        | 説明                             |
|---------------|-----------|----------------------------------|
| id            | UUID      | 主キー                           |
| name          | TEXT      | nestの名前（例：家族・推し活 etc） |
| color_theme   | TEXT      | UI表示用の色テーマ識別子         |
| icon          | TEXT      | nest用アイコン（絵文字 or URL） |
| created_by    | UUID      | 作成者（users.id）               |
| is_private    | BOOLEAN   | 招待制かどうか                   |
| created_at    | TIMESTAMP | 作成日時                         |

---

## 👥 nest_members テーブル（中間テーブル）

| カラム名    | 型    | 説明                    |
|-------------|-------|-------------------------|
| nest_id     | UUID  | nests.id                |
| user_id     | UUID  | users.id                |
| joined_at   | TIMESTAMP | 参加日時           |
| is_muted    | BOOLEAN | 通知ミュートフラグ（任意） |

- 複合主キー：`(nest_id, user_id)`

---

## 📩 piins テーブル（piinの送信ログ）

| カラム名    | 型        | 説明                                 |
|-------------|-----------|--------------------------------------|
| id          | UUID      | 主キー                               |
| sender_id   | UUID      | 送信者（users.id）                   |
| nest_id     | UUID      | 宛先のnest（nests.id）               |
| tone        | TEXT      | 気持ちのトーン（"calm" / "warm" など） |
| icon        | TEXT      | スタンプや感情記号（任意）           |
| sent_at     | TIMESTAMP | 送信日時                             |
| expires_at  | TIMESTAMP | 表示期限（例：5分後に自動で消す）     |

---

## 💬 reactions テーブル（任意のリアクション）

| カラム名    | 型        | 説明                     |
|-------------|-----------|--------------------------|
| piin_id     | UUID      | piins.id                 |
| user_id     | UUID      | 反応したユーザー         |
| emoji       | TEXT      | 絵文字リアクション       |
| reacted_at  | TIMESTAMP | リアクションした日時     |

- 複合主キー：`(piin_id, user_id)`

---

## 🔐 備考

- `piins`は「表示期間を短く」するため、`expires_at`を使って自動削除対象にする
- `nest_members`は実質的に「今いるかどうか」だけ持てれば良い
- `reactions`もMVPでは省略可能。演出だけで返すのもアリ

---

## スキーマ

```sql
-- schema.sql

-- schema.sql（ON DELETE CASCADE 対応）

-- 必須：UUID生成に必要
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- ユーザーテーブル
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nickname TEXT NOT NULL,
    avatar_url TEXT,
    fcm_token TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT now(),
    updated_at TIMESTAMP NOT NULL DEFAULT now()
);

-- nestテーブル（作成者が削除された場合はNULLに）
CREATE TABLE nests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    color_theme TEXT,
    icon TEXT,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    is_private BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT now()
);

-- nestメンバー（ユーザーが削除されたら自動削除）
CREATE TABLE nest_members (
    nest_id UUID NOT NULL REFERENCES nests(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    joined_at TIMESTAMP NOT NULL DEFAULT now(),
    is_muted BOOLEAN NOT NULL DEFAULT FALSE,
    PRIMARY KEY (nest_id, user_id)
);

-- piin送信ログ（ユーザーまたはnestが削除されたら連鎖削除）
CREATE TABLE piins (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sender_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    nest_id UUID NOT NULL REFERENCES nests(id) ON DELETE CASCADE,
    tone TEXT,
    icon TEXT,
    sent_at TIMESTAMP NOT NULL DEFAULT now(),
    expires_at TIMESTAMP
);

-- リアクション（ユーザーまたはpiinが削除されたら連鎖削除）
CREATE TABLE reactions (
    piin_id UUID NOT NULL REFERENCES piins(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    emoji TEXT,
    reacted_at TIMESTAMP NOT NULL DEFAULT now(),
    PRIMARY KEY (piin_id, user_id)
);
```
