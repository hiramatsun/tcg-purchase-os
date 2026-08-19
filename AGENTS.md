# TCG Purchase OS agent instructions

## Mission

日本国内の正規販売ルートにおけるTCGの購入機会を発見・検証・整理し、ユーザーが次に取るべき行動を判断できる状態をつくる。

初期対象は Pokémon Card Game、ONE PIECE CARD GAME、DRAGON BALL SUPER CARD GAME Fusion World とする。

## Rules

- 回答と運用文書は、ユーザーが別言語を指定しない限り日本語で記述する。
- 日時は `Asia/Tokyo` を基準にし、保存時は ISO 8601 と UTC オフセットを含める。
- 重要な事実は実際に開いた根拠で確認する。検索スニペットだけで確定しない。販売速報は店舗公式SNSまたは現地目撃投稿も対象にするが、投稿時点のシグナルとして扱い、現在在庫を断定しない。
- 確認できない値は推測せず `null` または未確認として扱う。
- 各購入機会に `source_url`、`source_type`、`last_verified_at`、`confidence`、`status` を持たせる。
- 抽選機会には、応募経路を表す `lottery_method` と、当選・購入後の受け取り方法を表す `fulfillment_method` を別々に持たせる。`channel` は互換性のため残すが、これらの代用にしない。
- 発売日販売・通常販売・再販は抽選と別の購入機会として扱い、正式告知がある場合は `sale_announced_at` と販売主体の一次情報を記録する。店舗公式SNS・現地目撃の入荷速報は `sale_signal_at`、投稿者種別、在庫シグナルを別に記録し、告知時点・投稿時点と現在の在庫状態を混同しない。
- 一次情報で抽選・販売・再販などを確認した販売主体や支店は `data/retailers.yaml` に候補として蓄積し、Opportunity の `retailer_id` で参照する。候補の蓄積は網羅性を意味しない。
- メーカー公式、販売主体の公式告知、正規小売店の順に優先し、二次情報だけで重要な判断を確定しない。
- オンライン応募・配送は全国の販売主体を調査対象にし、店頭応募・店頭販売・店頭受取は `data/preferences.yaml` の基準地点から公共交通機関で片道上限以内に限定する。オンライン応募後の店頭受取は、応募範囲と受取範囲を分けて確認する。
- 情報源の利用規約、robots、アクセス制限を尊重し、認証・技術的制限を迂回しない。
- 自動購入、抽選の自動応募、複数アカウント運用、決済操作は行わない。最終操作は人が実行する。
- 終了機会や訂正前の値を削除して履歴を失わない。状態変更と根拠を記録する。
- 秘密情報、認証情報、個人情報をリポジトリへ保存しない。

## Standard workflow

1. `data/preferences.yaml` と `data/sources.yaml` で対象と情報源を確認する。
2. 情報源を開き、商品、販売主体、機会種別、期間、価格、条件、URL、確認日時を抽出する。
3. `docs/data-model.md` に従って正規化し、既存データと比較する。
4. 新規、変更、終了、未確認を判定し、根拠と変更履歴を保存する。
5. `docs/purchase-rules.md` に従って、今日必要な行動を優先順位付きで報告する。
6. ユーザーの応募・購入結果は、機会の事実状態と混同せずイベントとして記録する。

## Repository skills

- `$research-tcg-opportunities`: 購入機会を調査・検証する。
- `$maintain-tcg-data`: 調査結果を正規化し、名寄せ・差分反映する。
- `$report-tcg-actions`: 締切と状態から今日の行動を作る。
- `$record-tcg-outcomes`: 応募、当落、購入などの結果を記録する。

調査だけではファイルを書き換えず、「保存」「更新」「同期」まで依頼された場合だけデータを更新する。

## File map

- `README.md`: Mission、範囲、ロードマップ
- `.agents/skills/`: 再利用可能な Codex ワークフロー
- `docs/data-model.md`: 保存形式とフィールド定義の正本
- `docs/purchase-rules.md`: 信頼度、鮮度、優先順位、禁止事項
- `data/sources.yaml`: 監視情報源レジストリ
- `data/retailers.yaml`: 一次情報で購入機会が確認できた販売主体・店舗候補と調査範囲
- `data/preferences.yaml`: 対象タイトルと希望条件
- `data/products.json`: 商品レコード
- `data/opportunities.json`: 購入機会と変更履歴
- `data/outcomes.json`: ユーザー行動・結果の追記型ログ

## Data editing

- JSON は UTF-8、2スペースインデント、末尾改行で保存する。
- トップレベルの `schema_version` を維持する。
- ID は作成後に変更・再利用しない。
- `first_seen_at` を保持し、`last_verified_at` と `updated_at` を適切に更新する。
- 根拠のない追加、状態変更、締切補完を行わない。
- 変更した JSON は `python3 -m json.tool <file>` で検証する。
- Skill は `skill-creator` の `quick_validate.py` で検証する。

コミットや push はユーザーが明示的に依頼した場合だけ行う。
