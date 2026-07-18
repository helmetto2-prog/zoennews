# 会社PCのClaudeへ：バックナンバー運用のお願い（2026-07-18 自宅PCより）

2026-07-18に自宅PCのClaudeが、Git履歴から過去12号を復元して
`backnumber/` フォルダにバックナンバーを整備しました。

- 一覧ページ: `backnumber/index.html`（公開URL: https://helmetto2-prog.github.io/zoennews/backnumber/ ）
- 各号: `backnumber/YYYY-MM-DD.html`（紙面に記載の日付でファイル名を付ける）

## 毎朝5時の自動更新に追加してほしい2ステップ

1. **その日の新しい index.html を `backnumber/YYYY-MM-DD.html` にもコピーして追加する**
   （末尾に「📚 バックナンバー一覧へ戻る」リンク付き。既存号の末尾を参照）
2. **`backnumber/index.html` の一覧の先頭に新しい号のカードを追記する**
   （日付・曜日・見出しリスト・リンク。既存カードの構造をコピーして使う。
   ヘッダーの「全N号」のカウントも+1する）

また、index.html（最新号）の `</body>` 直前に入れている
「📚 バックナンバー一覧（全N号）」リンクを、毎日の生成テンプレートにも含めてください
（今のままだと翌朝の上書きでリンクが消えます）。

以上を組み込んだら、このNOTEは手順書（SKILL_取引先ニュース贈呈_手順書.md ではなく
zoennews側の手順メモ）に反映のうえ削除してかまいません。
