# Data model

## Conventions

- 文字コードは UTF-8、JSON は2スペースインデントとする。
- 日時は ISO 8601 形式で UTC オフセットを含める。例: `2026-08-19T09:00:00+09:00`。
- 日付だけが公表されている値は `YYYY-MM-DD` とし、時刻を推測しない。
- 金額は税込・円で確認できる場合だけ整数で保存する。
- 確認できない値は空文字ではなく `null` にする。
- すべての ID は一度作成したら変更・再利用しない。

情報源に安定した公開 ID があればそれを含める。ない場合は、ゲーム、商品または販売主体、機会種別、開始日を kebab-case で結合する。衝突時だけ短い連番を付ける。

## `products.json`

トップレベルは `{ "schema_version": 1, "products": [] }` とする。各 Product は `id`、`game`、`name`、`product_type`、`release_date`、`price_jpy`、`official_url`、`status`、`first_seen_at`、`last_verified_at`、`notes` を持つ。

`game` は `pokemon-card-game`、`one-piece-card-game`、`dragon-ball-super-card-game-fusion-world` のいずれか。`product_type` は `booster-pack`、`starter-deck`、`deck`、`box`、`set`、`accessory`、`promo`、`other` のいずれか。`status` は `announced`、`released`、`discontinued`、`unconfirmed` のいずれかとする。

## `opportunities.json`

トップレベルは `{ "schema_version": 1, "opportunities": [] }` とする。各 Opportunity は次のフィールドを持つ。

| Field | Meaning |
| --- | --- |
| `id` | 安定した購入機会 ID |
| `product_id` | `products.json` の ID。特定前は `null` |
| `game`, `title` | タイトルと機会名 |
| `opportunity_type` | `preorder`、`lottery`、`made-to-order`、`restock`、`release-day-sale`、`general-sale`、`announcement` |
| `seller` | 販売・抽選の実施主体 |
| `channel` | `online`、`store`、`app`、`mixed`、`unknown` |
| `status` | `upcoming`、`open`、`ended`、`cancelled`、`unconfirmed` |
| `starts_at`, `ends_at` | 受付期間。固定締切なしは `ends_at: null` |
| `result_announced_at` | 抽選結果発表 |
| `price_jpy`, `eligibility` | 価格と条件 |
| `action_required` | ユーザーの次の操作 |
| `source_url`, `source_type` | 主な根拠 URL と種別 |
| `last_verified_at`, `first_seen_at`, `updated_at` | 観測・更新日時 |
| `confidence` | `high`、`medium`、`low` |
| `notes` | 注意事項、競合情報 |
| `evidence` | `{url, source_type, observed_at, claim}` の配列 |
| `changes` | `{observed_at, field, previous, current, source_url}` の配列 |

`source_type` は `official-publisher`、`official-retailer`、`official-social`、`secondary`、`unknown` のいずれかとする。

## `outcomes.json`

トップレベルは `{ "schema_version": 1, "events": [] }` とする。各イベントは `id`、`opportunity_id`、`event_type`、`occurred_at`、`quantity`、`amount_jpy`、`note`、`created_at` を持つ。`event_type` は `applied`、`reserved`、`won`、`lost`、`purchased`、`skipped`、`cancelled`、`refunded` のいずれかとする。

注文番号、決済情報、住所、電話番号、アカウント ID などの個人情報は保存しない。

## Deduplication

Product はゲーム、正規化した商品名、商品区分、発売日を比較する。Opportunity は商品、販売主体、機会種別、受付期間、主な根拠 URL を比較する。表記差だけなら統合し、販売主体、応募条件、期間が実質的に異なる場合は別レコードにする。判断できない場合は自動統合せず `notes` に候補 ID を残す。

