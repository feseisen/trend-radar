# 05 図を作るスキル（diagram-design と archify）

号: 2026-10-17 ／ 判定: **採用**（定番入り） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: 頼むと、**ネット無しでも開ける HTML 1枚の図** を作るスキル。2つあり、得意分野が違う
  - **diagram-design**（[リポジトリ](https://github.com/cathrynlavery/diagram-design)、MIT）: 構成図・シーケンス図・フローチャート・状態図・ER図・年表・スイムレーン・4象限・ガントチャート・サンキー図など 40 種類以上。雑誌のような落ち着いた見た目。draw.io・Mermaid・Excalidraw の図を描き直せる
  - **archify**（[リポジトリ](https://github.com/tt-a1i/archify)、MIT）: 構成・業務フロー・シーケンス・データフロー・状態遷移の5種類を、**触って操作できる図** にする。コードのリポジトリを読ませて構成図を作れる
- **なぜいま話題か**: どちらも10月に GitHub で大きく伸びた（集計サイトで +13.1k と +8.8k）。合計のスターは 56.3k と 83.4k（2026-10-17 にリポジトリのページで見た値）
- **取り入れると何が良くなるか**（推測）: 資料の図を作る時間が減る。説明会や設計の打ち合わせの図を、その場で直せる
- **手間**: 入れるのはコマンド1行
- **気をつけること**: diagram-design の図は、既定で Google Fonts を読みに行く（それが唯一の外への通信。システムフォントに切り替える設定がある）。archify は更新の確認で外へ1回読みに行く（オフにできる）。図の中身に社外秘が入るなら、外へ出さない置き場で扱う

### どちらを使うか（推測による目安）

| こうしたいなら | 使う |
|---|---|
| 資料に貼る、見た目の整った図。種類が多い | diagram-design |
| 説明の場で触って見せる図。コードから構成を起こす | archify |
| claude.ai で使いたい | archify（claude.ai にアップロードする形を用意している） |
| 頻繁に直して Git で差分を見たい | どちらでもなく Mermaid（diagram-design の README 自身がそう書いている） |

### 対応環境

| | diagram-design | archify |
|---|---|---|
| 使える AI | Claude Code、Codex、GitHub Copilot、Cursor ほか多数 | Claude Code、Codex CLI、Cursor、OpenCode。claude.ai（zip をアップロードする） |
| 要るもの | PNG に書き出すときだけ Python の playwright | Node.js だけ（ほかに入れるものは要らない、と README にある） |
| ネット | 図を開くと Google Fonts を読み込む（オフにできる） | 更新の確認（オフにできる）と、図の中の外部のサイトや地図へのリンクを開くとき |
| API キー | 要らない | 要らない |
| ライセンス | MIT | MIT |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「図にして」と頼んだら、上の「どちらを使うか」で選び、入っていなければ入れて（04 のチェックをしてから）、図を **HTML 1枚** で作ってください。

### diagram-design

入れ方（README のとおり。どれか1つ）:

```bash
npx skills add cathrynlavery/diagram-design
```

```
/plugin marketplace add cathrynlavery/diagram-design
```

```
/plugin install diagram-design@diagram-design
```

```bash
codex plugin marketplace add cathrynlavery/diagram-design
codex plugin add diagram-design@diagram-design
```

頼み方の例（README の例。日本語で頼んでよい。図の中の文字の言語を指定する）:

- 「私のアプリの構成図を作って: フロントエンド、バックエンド、データベース、Redis のキャッシュ」
- 「第2四半期のプロジェクトを、効果と手間の4象限に並べて」
- 「トークンの期限切れ（401）で更新する呼び出しのシーケンス図を作って」
- 「https://（会社のサイト）に合わせて」と頼むと、そのサイトの色とフォントを取り込んで図の見た目を合わせる

出力と書き出し:

- 出力は `.html` 1枚。ダブルクリックでネット無しでも開ける
- SVG への書き出しはスキルに付属のスクリプト。PNG（2倍の解像度）は playwright が要る:

```bash
pip install playwright
playwright install chromium
```

- 図は既定で Google Fonts を読みに行く。外へ出られない端末や、外へ通信させたくない場合は、システムフォントを使う設定で作るよう頼む（README に「system-fonts」の選択肢があると書かれている）

注意（README の記載）:

- 同じ依頼でも、作るたびに配置が変わることがある
- できた HTML は編集用の元データではないので、Git の差分は読みにくい。頻繁に直す文書の図は Mermaid のほうが向く

### archify

入れ方（README のとおり）:

```bash
npx skills add tt-a1i/archify -g
```

Codex なら:

```bash
npx skills use tt-a1i/archify@archify --agent codex
```

claude.ai なら: リポジトリの `archify.zip` を「カスタマイズ → スキル」からアップロードする（入れ方は 04）。README は「claude.ai の実行環境（サンドボックス）で Node.js が使えるかによる」と注意している。プロジェクトナレッジ（Project Knowledge）にアップロードして、指示文として使う形も書かれている。

頼み方の例（README の例）:

- 「archify で Web の呼び出しの図を: ブラウザが API を呼び、API が Redis を見て、無ければ PostgreSQL を引いてキャッシュを埋める」
- 「このリポジトリを読んでから、archify で動いているときの構成図を作って」。README はこのとき「中心の部品は 8〜12 個、主な道筋は1本」に絞るよう勧めている
- 作った後の直し: 「Redis を足して」「認証を左へ」「戻すときの道筋を目立たせて」

archify のコマンド（README のとおり。リポジトリを手元に置いた場合に、そのルートフォルダで実行する）:

```bash
node bin/archify.mjs doctor
```

`doctor` のほか、`demo`、`guide`、`validate`、`preview`、`deliver` がある。`preview` は手元（127.0.0.1）だけで開く。

更新の確認の通信を切る:

```bash
export ARCHIFY_UPDATE_CHECK_DISABLED=1
```

Windows の PowerShell なら:

```powershell
$env:ARCHIFY_UPDATE_CHECK_DISABLED = "1"
```

できないこと（README の記載）: 何でも描ける作図ソフトではない。Mermaid の読み込み、自動レイアウト、共有の置き場、見たまま直す編集は範囲の外。

### どちらでも守ること

- 図の中の文字は、利用者の言語に合わせる
- 社外秘の構成を描いたときは、図を外の共有の場（スキルの作例の投稿先など）へ出さない
- できた図を利用者に渡すとき、「何を描いたか」と「描いていないもの（省いた部品）」を一行ずつ添える

### 出典（2026-10-17 に読んだもの）

- [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design)（スター 56.3k、フォーク 3.6k、MIT）
- [tt-a1i/archify](https://github.com/tt-a1i/archify)（スター 83.4k、フォーク 5.6k、MIT、v3.1.0 のリリースは 2026-10-14）
