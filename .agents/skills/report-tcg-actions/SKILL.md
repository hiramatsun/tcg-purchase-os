---
name: report-tcg-actions
description: 保存済みのTCG商品、購入機会、ユーザー行動、希望条件から、締切・鮮度・信頼度・対応状況を評価し、「今日やること」を優先順位付きで作成する。ユーザーが日次レポート、今日の応募・予約・購入判断、締切一覧、未対応機会、要再確認情報を求めたときに使う。新しいWeb調査やデータ更新そのものには使わない。
---

# Report TCG Actions

## Load state

1. `docs/purchase-rules.md` と `docs/data-model.md` を読む。
2. `data/preferences.yaml`、`data/products.json`、`data/opportunities.json`、`data/outcomes.json` を読む。
3. 基準時刻を `Asia/Tokyo` で確定する。
4. 各 Opportunity に対する最新イベントを確認し、応募済み、予約済み、当落、購入済みを導出する。

レコードがない場合は、データが空であることを明示する。Web上に機会が存在しないとは断定しない。

## Evaluate

1. 鮮度目標を超えた機会を `要再確認` にする。
2. `open` と `upcoming` を中心に締切までの残り時間を計算する。抽選は未発売かつ発売日が近い順、販売速報は `sale_signal_at` の新しい順に扱う。
3. `wanted_products` とゲーム優先度を反映する。
4. 応募済み・購入済みは、結果確認や支払期限など追加行動がない限り除外する。
5. `confidence: low` は行動確定ではなく確認タスクとして扱う。
6. `ends_at: null` の先着・再販は、最終確認時刻と在庫根拠を明示する。X等の販売速報は `sale_signal_at`、`availability_signal`、`signal_source_kind` を表示し、投稿時点の目撃を現在在庫と混同しない。
7. P0、P1、P2、P3 の順に、同じ優先度では締切、希望度、信頼度で並べる。

## Output

日本語で次の構成を使う。

### 今日やること

P0 と P1 を並べ、商品、販売主体、`retailer_id`、必要操作、締切と残り時間、ユーザー状態、信頼度、最終確認日時、根拠 URL を含める。オンライン応募・配送は全国対象として表示し、店頭応募・店頭販売・店頭受取は `reachability` と60分上限内かどうかを表示する。発売・販売機会では `sale_announced_at`、`starts_at`、一次情報URL、告知時点の在庫・売切れ表示も含める。

### 近日中

P2 と受付開始前の準備事項を並べる。

### 監視・要再確認

P3、古い情報、未確認条件、低信頼情報を並べ、何を再確認すれば判断できるかを示す。販売速報は店舗名、投稿時刻、商品名、画像の有無、売切れ報告の有無を確認項目にする。

### 集計

受付中、受付前、終了、未対応、応募済み、購入済み、要再確認の件数を示す。

レポート作成だけではファイルを変更しない。新鮮な調査が必要なら `$research-tcg-opportunities`、結果保存が必要なら `$record-tcg-outcomes` を使う。
