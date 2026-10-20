# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

「ヴェネツィア商人の航路開拓 / Venetian Merchant's Route」。15世紀ヴェネツィアを舞台に、AI商人ダンドロと1対1で対戦する戦略カードゲーム。日英2言語、スマホ縦画面前提（maxWidth 520）。作者表記は TANEGAMES。

- 公開先: GitHub Pages https://shigetane2011-cell.github.io/venetian-merchant/ （main ブランチ直下の index.html をそのまま配信）
- ビルド工程なし。`index.html` 1枚にすべて入っている（React 18 UMD + Babel standalone を cdnjs から読み込み、`<script type="text/babel">` 内に JSX を直書き）
- npm / テストランナー / Lint は無い。動作確認はブラウザで index.html を開くか、公開ページで行う
- 現在の表示バージョンは v2.2（index.html 内の2か所に文字列として埋め込み。更新時は両方直す）

## ゲームルール（index.html の TEXTS.rules と一致させること）

- カードは方角4種（N/E/S/W）×1〜10 の40枚。初期手札7枚、残り26枚が山札
- 手番: 手札1枚を選び「配置 / 重点契約 / 捨てる」のいずれか。配置・捨てたら山札から1枚補充
- 配置: その航路の自分の最大値より大きい数字のみ。「1」は空の航路にのみ出せ、相手の同じ航路の最大カードを奪って手札に加える（奪ったときは補充なし）
- 重点契約: 自分の空の航路に宣言。得点2倍、ただし −20 のペナルティ。手札は消費せず補充もしない
- 山札が尽きたら終了。航路ごとの合計点の総和で勝敗
- 貿易風イベント（タイトルでON/OFF、ゲームごとにランダム1つ）: 凪=契約ペナルティ −10 / 海賊横行=「1」で強奪成功時も1枚補充 / 市祭=3枚以上置いた航路に +5

## ファイル

- `index.html` — ゲーム本体
- `ogp.png` — OGP画像 1200x630。Pillow スクリプトで生成したもの（スクリプトはリポに含めていない）
- `CLAUDE.md` — このファイル

## index.html の構成（行番号は目安）

| 範囲 | 内容 |
|---|---|
| 先頭〜65 | メタ情報・OGP・フォント・keyframes・CDN script。Google Analytics はコメントアウトのまま（測定ID未設定の TODO） |
| 70〜236 | 定数と `TEXTS`（ja/en の全文言。文言追加は必ず両言語） |
| 238〜277 | `SFX` Web Audio による効果音（外部ファイル無し） |
| 279〜299 | localStorage 読み書きと `updateRank`（100戦ごとの期間別成績、ベスト3連戦） |
| 301〜379 | エンジン: `initGame` / `processTurn` / `calcScoreEx` / `totalScore` など純粋関数 |
| 381〜693 | ダンドロAI: `aiDecideEasy`（わざと悪手）/ `aiDecideStd`（契約→強奪→終盤最適化→死に札最小化）/ `aiDecideHard`（モンテカルロ、同一48通りの配りで候補手を比較） |
| 695〜707 | `getDandolo` セリフと表情 |
| 709〜777 | 小コンポーネント（MiniCard / HandCard / DirectionRow / StatsBar / Confetti / DandoloBar） |
| 779〜1055 | `App`（タイトル画面・ゲーム画面・実績/記録/遊び方モーダル） |
| 1058 | `window.__VM` にエンジンと `aiDecide` を公開（自動テスト用） |

## localStorage キー

`vm_lang` `vm_diff` `vm_event` `vm_muted` `vm_stats`(直近10戦) `vm_career`(通算) `vm_ach`(実績ID配列) `vm_rank`(期間別成績)。形式を変えるときは `loadLS` 側で旧形式を弾く分岐を入れてあるので、それに倣う。

## 動作確認の方法

公開ページまたはローカルで index.html を開き、ブラウザのコンソールで `window.__VM` を使ってAI同士を回すと、UIなしで1ゲームを最後まで検証できる。`turnPlayer` が `"user"` の番は aiHand/userHand などを入れ替えた状態を `aiDecide` に渡せば、両者をAIにできる。UI の主要ボタンには `data-tid` 属性が付いている（start / hand-card / act-place / act-contract / act-discard / contract-dir / again / to-title など）。

## 既知の未対応

- Google Analytics の測定ID未設定
- README が無い（LICENSE は MIT）
- 難易度「難しい」はモンテカルロだが、「普通」との AI 対戦で勝率が 5〜6割程度にとどまる（2026-10-20 に20戦で確認）。強さの差が小さい
