---
name: record-tcg-outcomes
description: ユーザーが行ったTCGの抽選応募、予約、当選、落選、購入、見送り、取消、返金を、購入機会に紐づく追記型イベントとして安全に記録する。ユーザーが「応募した」「当選した」「購入した」などの結果保存や履歴更新を依頼したときに使う。販売機会そのものの状態更新やWeb調査には使わない。
---

# Record TCG Outcomes

## Resolve the opportunity

1. `docs/data-model.md`、`data/opportunities.json`、`data/outcomes.json` を読む。
2. ユーザーの説明から対象 Opportunity を特定する。
3. 候補が複数ある場合は、販売主体、商品、受付期間を示して特定を求める。誤った機会へ推測で紐づけない。
4. 対象 Opportunity がない場合は、根拠なしに作成せず、先に調査またはデータ追加が必要と伝える。

## Normalize the event

次の表現を対応する `event_type` に変換する。

- 応募した: `applied`
- 予約した: `reserved`
- 当選した: `won`
- 落選した: `lost`
- 購入した: `purchased`
- 見送った: `skipped`
- 取消した: `cancelled`
- 返金された: `refunded`

`occurred_at` は指定日時を使う。「今日」「さっき」は `Asia/Tokyo` の現在時刻を基準に解決し、解決できなければ確認する。数量と総額は示された場合だけ保存し、不明なら `null` にする。

注文番号、決済情報、住所、電話番号、アカウント ID は `note` に保存しない。

## Append safely

1. ID は `evt-{opportunity_id}-{occurred_atの基本形式}-{event_type}` を基にし、衝突時だけ連番を付ける。
2. 既存イベントを上書きせず `events` に追記する。
3. 同一機会・種別・発生日時のイベントがあれば重複を確認する。
4. `events` を `occurred_at`、次に `id` で並べる。
5. JSON を2スペースインデントと末尾改行で保存し、`python3 -m json.tool data/outcomes.json` で検証する。

ユーザー結果を理由に Opportunity の `status` を変更しない。販売機会の終了や取消は、販売主体の根拠を確認して `$maintain-tcg-data` で更新する。

## Confirm

記録後は、商品、販売主体、イベント種別、発生日時、数量、総額、Opportunity ID を日本語で簡潔に報告する。個人情報は表示しない。
