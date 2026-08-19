---
name: maintain-tcg-data
description: 根拠付きのTCG商品・購入機会候補を共通スキーマへ正規化し、名寄せ、重複排除、新規・変更・終了判定、変更履歴の追記、JSONデータ更新を行う。ユーザーが調査結果の保存、同期、データ更新、重複整理、既存データとの差分反映を依頼したときに使う。Web調査だけ、日次レポートだけ、ユーザー行動の記録だけには使わない。
---

# Maintain TCG Data

## Load contracts

1. `AGENTS.md` と `docs/data-model.md` を読む。
2. `data/products.json`、`data/opportunities.json`、必要に応じて `data/sources.yaml` を読む。
3. 入力候補の根拠 URL、確認日時、信頼度を確認する。根拠がない事実は保存しない。

## Normalize and match

- 公式の商品名と販売主体名を優先する。
- 列挙値、日時、金額、`null` の扱いをデータモデルに合わせる。
- 既存 ID を変更しない。新規 ID はデータモデルの規則で作る。
- Product はゲーム、正規化した商品名、商品区分、発売日を比較する。
- Opportunity は商品、販売主体、機会種別、受付期間、根拠 URL を比較する。
- 表記差だけなら統合し、販売主体、条件、期間が実質的に異なる場合は別レコードにする。
- 判断できない候補を自動統合せず、候補 ID を `notes` に残す。

## Apply changes

### New records

- 必須フィールドをすべて設定する。
- `first_seen_at`、`last_verified_at`、`updated_at` を観測時刻から設定する。
- Opportunity に Evidence を最低1件付ける。

### Existing records

- `first_seen_at` を保持する。
- 再確認では `last_verified_at` を更新し、新しい Evidence を必要に応じて追加する。
- 事実値が変わった場合は値を更新し、フィールドごとに Change を追記する。
- 一次情報を弱い根拠で上書きせず、競合を `notes` に残す。
- 終了・取消も削除せず `status` を変更する。

### Stable output

- JSON は UTF-8、2スペースインデント、末尾改行で保存する。
- 意味上の順序がない配列は `id` で並べる。Evidence と Change は時系列で並べる。
- 無関係な整形変更を含めない。

## Validate and summarize

1. 変更した JSON を `python3 -m json.tool` で解析する。
2. 参照整合性、列挙値、必須フィールド、ID の一意性を確認する。
3. Git 差分を読み、根拠のない補完や履歴消失がないことを確認する。
4. 新規、更新、終了、変更なし、要確認の件数と ID を日本語で報告する。

ユーザーの応募・購入結果は `opportunities.json` に混ぜず、`$record-tcg-outcomes` で `outcomes.json` に追記する。
