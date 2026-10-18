# 週刊 みどりの造園ニュース ｜ 金曜の発行手順書（クラウドの定期実行用）

毎週金曜 18:12（日本時間）の定期実行は、この手順書に従って作業すること。確認は不要。静かに最後まで完走する。
（2026-10-18 ゲン作成。元はまとんの「_ルーティン_金曜週刊まとめ.md」。第9号＝2026-10-23 からクラウドで発行）
（2026-10-18 まとんが今の作り方と見比べて直した：§1 の「今週分」の数え方と push 直前の見直し／§5 の絵の選び方）

## 0. 前提

- リポジトリ：helmetto2-prog/zoennews（main に直接 push。Pages が main を配信）
- 出力：`weekly/YYYYMMDD.html`（YYYYMMDD＝その金曜の日付・日本時間）
- 公開URL：`https://helmetto2-prog.github.io/zoennews/weekly/YYYYMMDD.html`
- 日付・曜日は必ず日本時間（JST＝UTC+9）で判断する
- git の名前・メールは変えない（コミットの作者は「Claude <noreply@anthropic.com>」のまま。自宅PCの保険がこれで見分ける）

## 1. 二重発行の防止（最初にやる）

1. `git pull --rebase`（無ければ clone）。
2. `weekly/` に、**今週の土曜〜今日（金曜）の日付**（＝6日前〜今日）のファイルが**もうあれば、何も作らずに終わる**。LINEも送らない。報告に「今週分は発行済みのため見送り」と1行。
   （自宅PCや手作業で先に出している場合があるため）
   ⚠ 先週の金曜（7日前）の号は「今週分」に数えない。例：2026-10-23 の回では `20261016.html` があっても作る。
3. **push の直前にもう一度** `git pull --rebase` して、今週の号が誰かに出されていないか見る。出ていたら、作った物は commit せずに捨てて見送る（自宅PCの保険が 19:30 に動く週があるため）。

## 2. 材料

- 対象：**土曜〜金曜の7日分**のデイリー `backnumber/YYYY-MM-DD.html`（その金曜朝5時の号まで）。見出しは `backnumber/index.html` が速い。
- 当日分が無ければ6日分でまとめ、報告に1行書く。
- 型と見本：`weekly/` の一番新しい号（2026-10-16＝第8号）。CSS・構成・ヘッダー・CTA・footer をそのまま使う。

## 3. 号数と記事番号

- 号数は既刊の一番新しい号＋1（2026-10-23 は第9号）。
- 記事番号（`<span class="no">`）も既刊の続きから連番（第8号の最後は30）。

## 4. 中身の決まり

- **一般のお客さん向け**。やさしい言葉、専門用語は `div.yaku`（📖）で言い換える。
- デイリーの単純な結合にしない。**週1本の読み物として独立**させる。
- **分量は創刊号の3割減**が目安（第8号くらい＝PART 1 に記事3〜4本＋PART 2 の企画1本＋CTA）。
- 出典リンク必須（`div.src`）。出典の取れない話題は書かない。断定しない。
- 地元（豊田・愛知）の季節の話題があれば PART 2「週末のおでかけ」に。開催日などは「変わることがある」旨を添える。
- **載せない物**：KY絵本の紹介・リンク／個人邸の写真／顧客名／金額（見積・単価）／社員の名前。
- footer に「社内向けデイリー造園ニュース（M月D日〜M月D日）を、お客様向けに再編集」「◯月◯日時点の公表内容」「イラストは自社制作です」を書く。

## 5. 絵（新しく描かない・KY絵本の挿絵を借りる）

- 絵は公開済みのKY絵本から借りる（tanpatsu-news の `ky-ehonNN.html`。1巻に挿絵11枚が base64 で埋め込まれている）。
- 取り方（全部をcloneしない。1巻ずつ取る）：
  ```bash
  git clone --depth 1 --filter=blob:none --sparse https://github.com/helmetto2-prog/tanpatsu-news tp
  cd tp && git ls-tree --name-only HEAD | grep '^ky-ehon[0-9]*\.html$'   # 巻の一覧
  git sparse-checkout set --no-cone /ky-ehon51.html                     # 使う巻だけ
  ```
- 挿絵は `src="data:image/...;base64,..."` をそのまま `<img class="photo">` に入れる（外部リンクにしない）。1号に3〜4枚。
- 記事の中身と絵が完全に一致していなくてよい（彩り・雰囲気）。`alt` と `caption` は**記事に合わせて自分で書く**（絵本の文は写さない）。
- **同じ絵を二度使わない**：`weekly/used_images.txt` に、使った絵の sha1（base64 の文字列で計算）・号のファイル・出どころ（巻）を1行ずつ足す。選ぶ前にこの記録と照らす。
  ```python
  import hashlib; hashlib.sha1(b64_string.encode()).hexdigest()
  ```
- 新しい巻から順に見て、まだ使っていない絵を選ぶ。
- **その52以降は ky-img/ の別ファイルから取る。catalog.json を見て選び、使った所を足す**（2026-10-18〜。KY絵本は1巻5話・絵は tanpatsu-news の `ky-img/<巻>_<話>.jpg`＝ページに base64 は無い。取り方＝`git sparse-checkout set --no-cone /ky-img/catalog.json /ky-img/52_01.jpg`。号には今まで通り base64 で入れ、sha1 もその base64 で計算して used_images.txt へ。catalog.json の その行の `used` に `"週刊YYYYMMDD"` を足して tanpatsu-news へ push（push の前に pull）。その51までの絵も ky-img/ にあるので、どの巻もここから取ってよい）
- **危ない場面（悪い手本）の絵は使わない**。KY絵本には「こうすると危ない」場面の絵が混じっている。お客さん向けの号に載ると、みどり造園が危ない作業をしているように見えるため、安全にできている場面・景色・道具の絵だけを選ぶ（まとんも挿絵ライブラリでそう選んでいた）。
- 選んだ絵は、絵の中に文字が描かれていないかも見る（文字のある絵は避ける）。
- 大きさの目安：KY絵本の挿絵は1枚およそ190〜320KB（base64）。4枚で約1MB＝1号はおよそ1.5MB以内に収まる。

## 6. 公開

1. `weekly/YYYYMMDD.html` を足す。**既刊（weekly・backnumber）は書き換えない・消さない**。
2. `weekly/used_images.txt` に今号の分を足す。
3. 同じコミットで main へ push。コミットメッセージ例：「週刊 みどりの造園ニュース 第9号を発行（自動）」。
   push 失敗時はネットワークエラーに限り 2s/4s/8s/16s で最大4回やり直す。
4. 公開の確認：この環境から github.io へは届かないことがあるので、GitHub API で Pages の反映を見る。
   `https://api.github.com/repos/helmetto2-prog/zoennews/pages/builds/latest` の status が `built` かつ commit が今回の push と同じになるまで、1分おきに最大10分待つ。
   届くなら公開URLの HTTP 200 も見る。10分で `built` にならなければ、LINEは送ったうえで報告に書く。

## 7. LINE（1通だけ）

- fx-race-line リポジトリの `notify.yml` を `mcp__github__actions_run_trigger`（workflow_dispatch, ref=main）で起動する。
- `inputs.message` の形（改行は `\n`。URLは行頭に単独で、前後に空白か改行を入れる。全角カッコで囲まない）：
  ```
  📰 週刊 みどりの造園ニュース 第N号\n\nhttps://helmetto2-prog.github.io/zoennews/weekly/YYYYMMDD.html \n\n（今週のいちばんの話題を1行）
  ```
- 3〜5行に収める。
- 送ったら `https://api.github.com/repos/helmetto2-prog/fx-race-line/actions/workflows/notify.yml/runs?per_page=1` を60秒後→必要ならさらに120秒後に見て、conclusion が success でなければ「（再送）」を付けて1回だけ再送し、その結果も確かめる。
- 見送り（§1）の時は送らない。

## 8. 報告

- 最後に、公開URLを単独の行で書き、号数・記事数・使った絵の数・LINEの結果を簡潔に。失敗したら原因を書く。

## 9. してはいけないこと

- 既刊の週刊・デイリーの書き換え・削除
- 新しく絵を描く・外部の写真を転載する
- KY絵本の紹介・リンク、個人邸の写真、顧客名、金額、社員の名前
- CSS・デザインの大幅な変更（提案は報告に書くだけ）
- git の名前・メールの変更
- LINEを2通以上送る（再送1回は除く）
