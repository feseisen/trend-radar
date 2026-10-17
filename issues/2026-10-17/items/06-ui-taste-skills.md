# 06 AIが作る画面の質を上げるスキル（impeccable と taste-skill）

号: 2026-10-17 ／ 判定: **採用**（定番入り） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: AIに画面（Web ページ・道具の画面）を作らせると、似たような「AIっぽい見た目」になりがち。それを直すためのスキルが2つ大きく伸びている
  - **impeccable**（[リポジトリ](https://github.com/pbakaus/impeccable)、Apache-2.0、作者 Paul Bakaus）: 24 の命令（設計の前の整理、点検、仕上げ、色・文字・余白の直し、動きの追加など）と、「AIが作りがちな見た目」を機械的に見つける 59 の規則
  - **taste-skill**（[リポジトリ](https://github.com/Leonxlnx/taste-skill)、MIT）: 頼みの文から見た目の方向を読み取り、「ばらつき・動き・密度」の3つのつまみで調整する。落ち着いた高級感、最小限、無骨、などの型も別に入っている
- **なぜいま話題か**: 10月の GitHub の伸びで +8.6k と +5.8k（集計サイト）。合計は 82.2k と 94.3k（2026-10-17 の現物）
- **取り入れると何が良くなるか**（推測）: 社内向けの小さな道具や報告のページが、作り直さずに「ちゃんとして見える」ところまで行く
- **手間**: 入れるのは1行
- **気をつけること**: **impeccable は既定で、設計の選択を外のサーバーへ送る**（作品名・ファイル・コードは送らない、と README は書いている）。切る命令がある（下）。taste-skill の既定の版（v2）は「試験中」

### どちらを使うか（推測による目安）

| こうしたいなら | 使う |
|---|---|
| 作る前の整理から仕上げの点検まで、手順ごと任せたい | impeccable |
| 指示を1つ貼るだけで、見た目の方向を変えたい。ChatGPT でも使いたい | taste-skill |
| まず機械で「AIっぽさ」を数えたい（AIを使わずに） | impeccable の CLI の検出だけ |

### 対応環境

| | impeccable | taste-skill |
|---|---|---|
| 使える AI | Claude Code、Cursor、Codex、Gemini CLI、GitHub Copilot ほか多数 | Claude Code、Codex、Cursor ほか。`SKILL.md` を ChatGPT に貼ってもよい（README の記載） |
| 要るもの | Node.js。URL を点検するなら手元の Chrome・Chromium・Edge | 特に無い（貼るだけなら） |
| ネット | 入れるとき。手元の実行ファイルを一度だけ落とすことがある。設計の選択の送信（既定で入、切れる） | 入れるときだけ |
| API キー | 要らない（CLI の検出は AI も鍵も要らない） | 要らない |
| ライセンス | Apache-2.0 | MIT |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が画面を作る・直すとき、上の「どちらを使うか」で選び、04 の点検をしてから入れて使ってください。**impeccable を入れたら、最初に送信を切るかを利用者に聞いてください。**

### impeccable

入れる（README の推奨。全部の道具に入る）:

```bash
npx impeccable install
```

更新:

```bash
npx impeccable update
```

Claude Code のプラグインとしてなら:

```
/plugin marketplace add pbakaus/impeccable
```

**送信を切る**（README のとおり。どれか1つ）:

```bash
npx impeccable telemetry off
```

環境変数なら `DO_NOT_TRACK=1` か `IMPECCABLE_NO_TELEMETRY=1`。プロジェクトの設定に `"telemetry": false` を書く形もある。
送っているもの（README の記載）: 設計を始めたときに、選んだ選択肢などを impeccable.style へ。作品名・ファイル・コード・`PRODUCT.md` は送らない、とある。

最初に1回:

```
/impeccable init
```

これで `PRODUCT.md`（何の製品か、誰が使うか）が書かれ、後の命令がそれを前提にする。

よく使う命令（`/impeccable <命令>` の形）:

| 命令 | すること |
|---|---|
| `shape` | 作る前に画面の構成を決める |
| `craft` | 構成を決めてから作り、見ながら直す |
| `critique` | 分かりやすさ・優先順位の点検 |
| `audit` | アクセシビリティ・速さ・画面の幅への対応の点検 |
| `polish` | 仕上げ |
| `typeset` / `layout` / `colorize` | 文字・余白・色の直し |
| `clarify` | 画面の文言を分かりやすく |
| `harden` | エラー・長い文字・多言語などの端の場合 |

`/impeccable pin <命令>` で `/audit` のような短い呼び名を作れる。

気をつけること（README の記載）:

- 手元の実行ファイルが入っていなければ、決まった版を `~/.impeccable/bin/` に一度だけ落とす
- 対応する道具では、画面のファイルを直すたびに検出の hook が動く。Claude Code では、AI の命令を断っても hook が実行ファイルを落とすことがある
- 「live」の形は手元にサーバーを立て、文言の直しを当てるときに `package.json` の `impeccable:manual-edit-validate` を利用者の権限で動かす。**動かす前にその中身を読む**
- 検出が何も見つけなくても、見た目やアクセシビリティが良いことの証明にはならない

### taste-skill

入れる（README のとおり）:

```bash
npx skills add https://github.com/Leonxlnx/taste-skill
```

既定の1つだけ:

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
```

前の版（v1）の振る舞いに頼っているなら:

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend-v1"
```

入っている型（README の記載）:

| スキル | 向き |
|---|---|
| `design-taste-frontend` | 既定（v2、試験中）。頼みから方向を読み、3つのつまみ（ばらつき・動き・密度）で調整 |
| `redesign-existing-projects` | 今ある画面を点検してから直す |
| `high-end-visual-design` | 落ち着いた高級感 |
| `minimalist-ui` | 最小限の、編集物のような画面 |
| `industrial-brutalist-ui` | 無骨で硬い見た目 |
| `full-output-enforcement` | 出力が途中で切れる・手抜きになるのを防ぐ |
| `image-to-code` | 参考の画像を作り、それに合わせて作る（画像を作れる AI が要る） |

ChatGPT などで使う: 使いたい型の `SKILL.md` を開き、中身を会話の最初に貼る。

### どちらでも守ること

- 直す前の画面を残す（比べられるように）
- 「AIっぽさを消す」のが目的で、利用者や組織の決まった見た目（社内の配色・ロゴの決まり）があるなら、そちらを優先する
- 直した点を、利用者が判断できる言葉で一行ずつ報告する（「見出しの大きさの段を3つから2つに減らした」など）

### 出典（2026-10-17 に読んだもの）

- [pbakaus/impeccable](https://github.com/pbakaus/impeccable)（星 82.2k、フォーク 4.9k、Apache-2.0）
- [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill)（星 94.3k、フォーク 6.4k、MIT。README に協賛の表示と、「作者の名前を使った無関係のトークンがある」という注意書きがある）
