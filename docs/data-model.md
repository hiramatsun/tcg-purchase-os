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

## `preferences.yaml`

`location` は任意の調査範囲設定で、`origin_name`、`origin_type`、`max_one_way_minutes`、`online_opportunities_scope`、`store_opportunities_scope`、`transit_modes`、`include_store_opportunities`、`include_store_pickup`、`notes` を持つ。`online_opportunities_scope` は `nationwide` または `configured-region`、`store_opportunities_scope` は `within_max_one_way_minutes` または `configured-region` とする。住所そのものは保存せず、駅などの基準地点名を使う。オンライン応募でも店頭受取になる場合は、受取場所に対して店頭範囲を適用する。

## `retailers.yaml`

トップレベルは `{ "schema_version": 1, "coverage": {}, "retailers": [] }` とする。購入機会の一次情報を出した販売主体・店舗を、後続調査の候補として蓄積する。店舗・支店ごとに到達性を判定するため、チェーン全体と特定支店は必要に応じて別レコードにする。

各 Retailer は `id`、`name`、`branch_name`、`retailer_type`、`games`、`official_urls`、`location`、`observed_opportunity_types`、`status`、`first_seen_at`、`last_verified_at`、`evidence`、`notes` を持つ。`retailer_type` は `chain`、`branch`、`online-retailer`、`specialty-store`、`other`、`status` は `candidate`、`active`、`inactive`、`unconfirmed` とする。`official_urls` には確認できた公式サイト、店舗ページ、抽選ページ、公式SNS、公式アプリの URL を記録し、二次情報の URL だけでは候補を確定しない。

`coverage.status` は `not-established`、`partial`、`broad` のいずれかとする。これは調査した範囲の自己評価であり、`broad` でも全国網羅を意味しない。タイトル、地域、販売形態、情報源種別、確認日時を `scope`、`assessed_at`、`notes` に残す。

## `opportunities.json`

トップレベルは `{ "schema_version": 1, "opportunities": [] }` とする。各 Opportunity は次のフィールドを持つ。

| Field | Meaning |
| --- | --- |
| `id` | 安定した購入機会 ID |
| `product_id` | `products.json` の ID。特定前は `null` |
| `game`, `title` | タイトルと機会名 |
| `opportunity_type` | `preorder`、`lottery`、`made-to-order`、`restock`、`release-day-sale`、`general-sale`、`announcement` |
| `seller` | 販売・抽選の実施主体 |
| `retailer_id` | `retailers.yaml` の販売主体・店舗 ID。未登録または特定不能時は `null` |
| `channel` | 互換性のための従来項目。`online`、`store`、`app`、`mixed`、`unknown` |
| `lottery_method` | 抽選への応募方法。`online`（Webフォーム等）、`store`（店頭受付）、`app`（公式アプリ）、`mixed`、`unknown`。抽選以外の機会は `null` |
| `fulfillment_method` | 当選・購入後の受け取り方法。`delivery`（配送）、`store-pickup`（店頭受取）、`event-site`（会場受取）、`mixed`、`unknown` |
| `status` | `upcoming`、`open`、`ended`、`cancelled`、`unconfirmed` |
| `starts_at`, `ends_at` | 受付期間。固定締切なしは `ends_at: null` |
| `sale_announced_at` | 販売主体の一次情報に記載された発売・販売告知の公開日時。`release-day-sale`、`general-sale`、`restock` では記録し、時刻がない場合は公表日だけを保存する。公開日時が確認できない場合は `null` とし、アクセスした日時は `last_verified_at` に記録する。抽選など販売告知を伴わない機会は `null` |
| `sale_signal_at` | 店舗公式SNSまたは現地目撃投稿で、入荷・販売が観測された日時。正式な発売告知日時とは別に記録する |
| `availability_signal` | 投稿時点の在庫シグナル。`reported-in-stock`、`reported-sold-out`、`unknown`。投稿だけで現在在庫を保証しない |
| `signal_source_kind` | 販売速報の投稿者種別。`store-official-social`、`eyewitness-social`、`secondary-repost`、`unknown` |
| `result_announced_at` | 抽選結果発表 |
| `price_jpy`, `eligibility` | 価格と条件 |
| `action_required` | ユーザーの次の操作 |
| `source_url`, `source_type` | 主な根拠 URL と種別 |
| `last_verified_at`, `first_seen_at`, `updated_at` | 観測・更新日時 |
| `confidence` | `high`、`medium`、`low` |
| `notes` | 注意事項、競合情報 |
| `reachability` | 店頭機会向けの任意情報。`origin`、`max_one_way_minutes`、`one_way_minutes`、`transit_mode`、`transfers`、`application_location`、`pickup_location`、`route_url`、`within_limit`、`checked_at` を持つ。オンラインのみの場合は `null`。応募場所と受取場所が異なる場合はそれぞれ記録する。 |
| `evidence` | `{url, source_type, observed_at, claim}` の配列 |
| `changes` | `{observed_at, field, previous, current, source_url}` の配列 |

`source_type` は `official-publisher`、`official-retailer`、`official-social`、`social-sighting`、`secondary`、`unknown` のいずれかとする。`social-sighting` は販売速報の観測根拠であり、正式な販売主体の一次告知とは区別する。

`lottery_method` と `fulfillment_method` は別々に判定する。例えば、Webで応募して店頭で受け取る場合は、それぞれ `online` と `store-pickup` になる。抽選方法または受け取り方法が確認できない場合は該当フィールドを `unknown` とし、`channel` だけから推測しない。

発売・販売機会（`release-day-sale`、`general-sale`、`restock`）は、抽選機会と同じ商品・販売主体であっても別 Opportunity として保存する。正式な発売・販売告知の `source_url`、`evidence` は販売主体・メーカーの一次情報（`official-publisher`、`official-retailer`、`official-social`）を優先する。販売速報は `social-sighting` として別の根拠を持てるが、`sale_signal_at` と `availability_signal` を必須にし、投稿時点のシグナルであって現在在庫の保証ではないことを `notes` に残す。`sale_announced_at` は実際の発売日や在庫の保証ではなく、正式告知が公開された時点を表す。ページを確認した日時は `last_verified_at`、根拠を確認した日時は `evidence[].observed_at` に分けて記録する。

## `outcomes.json`

トップレベルは `{ "schema_version": 1, "events": [] }` とする。各イベントは `id`、`opportunity_id`、`event_type`、`occurred_at`、`quantity`、`amount_jpy`、`note`、`created_at` を持つ。`event_type` は `applied`、`reserved`、`won`、`lost`、`purchased`、`skipped`、`cancelled`、`refunded` のいずれかとする。

注文番号、決済情報、住所、電話番号、アカウント ID などの個人情報は保存しない。

## Deduplication

Product はゲーム、正規化した商品名、商品区分、発売日を比較する。Opportunity は商品、販売主体、機会種別、受付期間、主な根拠 URL を比較する。抽選と発売日販売・通常販売・再販は機会種別が異なるため必ず別レコードにする。表記差だけなら統合し、販売主体、応募条件、期間が実質的に異なる場合は別レコードにする。判断できない場合は自動統合せず `notes` に候補 ID を残す。
