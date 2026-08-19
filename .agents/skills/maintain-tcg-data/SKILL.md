---
name: maintain-tcg-data
description: 根拠付きのTCG商品・購入機会候補を共通スキーマへ正規化し、名寄せ、重複排除、新規・変更・終了判定、変更履歴の追記、JSONデータ更新を行う。ユーザーが調査結果の保存、同期、データ更新、重複整理、既存データとの差分反映を依頼したときに使う。Web調査だけ、日次レポートだけ、ユーザー行動の記録だけには使わない。
---

# Maintain TCG Data

## Load contracts

1. `AGENTS.md` と `docs/data-model.md` を読む。
2. `data/products.json`、`data/opportunities.json`、`data/retailers.yaml`、必要に応じて `data/sources.yaml` を読む。
3. 入力候補の根拠 URL、確認日時、信頼度を確認する。根拠がない事実は保存しない。

## Normalize and match

- 公式の商品名と販売主体名を優先する。
- 列挙値、日時、金額、`null` の扱いをデータモデルに合わせる。
- 抽選と発売日販売・通常販売・再販は、同一商品・販売主体でも別 Opportunity として保持する。販売機会の `sale_announced_at` と一次情報 `source_url` のフィールドを欠落させず、公開日時が確認できない場合は `null` とする。
- 一次情報で確認できた販売主体・支店を Retailer として名寄せし、未登録なら `data/retailers.yaml` に候補を追加する。公式 URL と Evidence がない候補は `unconfirmed` とする。候補数だけで全国網羅を主張しない。
- 既存 ID を変更しない。新規 ID はデータモデルの規則で作る。
- Product はゲーム、正規化した商品名、商品区分、発売日を比較する。
- Opportunity は商品、販売主体、機会種別、受付期間、根拠 URL を比較する。
- Retailer は公式名称、支店名、所在地、公式 URL を比較する。同一チェーンでも支店が異なる場合は、店頭到達性を別に判定できるよう別レコードにする。
- 表記差だけなら統合し、販売主体、条件、期間が実質的に異なる場合は別レコードにする。
- 判断できない候補を自動統合せず、候補 ID を `notes` に残す。

## Apply changes

### New records

- 必須フィールドをすべて設定する。
- 新規 Retailer には `first_seen_at`、`last_verified_at`、`status: candidate`、一次情報の Evidence を設定する。
- `first_seen_at`、`last_verified_at`、`updated_at` を観測時刻から設定する。
- Opportunity に Evidence を最低1件付ける。

### Existing records

- `first_seen_at` を保持する。
- 再確認では `last_verified_at` を更新し、新しい Evidence を必要に応じて追加する。
- 事実値が変わった場合は値を更新し、フィールドごとに Change を追記する。
- 一次情報を弱い根拠で上書きせず、競合を `notes` に残す。
- 終了・取消も削除せず `status` を変更する。
- 販売終了・売り切れ表示は、販売告知と別の状態変化として記録し、`sale_announced_at` を現在の在庫確認時刻として上書きしない。

### Stable output

- JSON は UTF-8、2スペースインデント、末尾改行で保存する。
- 意味上の順序がない配列は `id` で並べる。Evidence と Change は時系列で並べる。
- 無関係な整形変更を含めない。

## Validate and summarize

1. 変更した JSON を `python3 -m json.tool`、YAML を YAML パーサーで解析する。
2. 参照整合性、列挙値、必須フィールド、ID の一意性を確認する。
3. Git 差分を読み、根拠のない補完や履歴消失がないことを確認する。
4. 新規、更新、終了、変更なし、要確認の件数と ID を日本語で報告する。

ユーザーの応募・購入結果は `opportunities.json` に混ぜず、`$record-tcg-outcomes` で `outcomes.json` に追記する。
