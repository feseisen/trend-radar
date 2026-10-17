# 04 スキルの入れ方（05〜08 の土台）

号: 2026-10-17 ／ 判定: **採用**（定番入り。05〜08 を入れるための共通の土台） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: AIエージェントに足す「スキル」は、**`SKILL.md` という指示書の入ったフォルダ** で、道具が違っても同じ形になっている。この号で取り上げたスキル（05〜08）は、どれもこの形で配られている
- **なぜいま話題か**: 10月に GitHub で伸びたリポジトリの上位に、スキル集が並んでいる（集計サイトの 10月の表で、AI 関係 27 件のうち、スキルそのもの、またはスキルを同梱するものが 10 件）。入れる道具も共通化が進み、`npx skills` という1つの命令で 70 以上のエージェントに入れられる（[skills CLI の README](https://cdn.jsdelivr.net/gh/vercel-labs/skills@main/README.md)）
- **取り入れると何が良くなるか**（推測）: 一度覚えれば、新しいスキルが出るたびに同じ手順で試せる
- **手間**: 1つ入れるのに1〜2分
- **気をつけること**: スキルは「指示書」だけでなく、**手元で動くスクリプトや、外へ送る通信を含むことがある**。入れる前に中身を見る（下の点検の表）

### 入れ方は使う道具で4通り

| 使う道具 | 入れ方 | 備考 |
|---|---|---|
| Claude Code | プラグインとして入れる、またはフォルダを置く | いちばん多く用意されている |
| ほかのエージェント（Codex、Cursor など） | `npx skills add` | Node.js が要る |
| claude.ai | 設定 → Capabilities → Skills で、`.skill` か zip を上げる | 対応しているスキルだけ |
| ChatGPT などのチャット | `SKILL.md` の中身を貼る | 複数ファイルのスキルは一部しか効かない |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「このスキルを入れて」と頼んだら、下の手順で **点検してから** 入れてください。点検で引っかかったものは、入れる前に利用者に報告して判断をもらってください。

### スキルとは

- スキルは、`SKILL.md` を入口に持つフォルダ。`SKILL.md` の先頭には YAML で `name` と `description` が書かれている（skills CLI の README の説明）
- 補助の文書（`references/` など）やスクリプト（`scripts/`）を同じフォルダに持つことがある

### 1. 入れる前の点検（必ず）

リポジトリを開き、次を見て、利用者に1つずつ報告する。

| 見るところ | 何を探すか |
|---|---|
| `SKILL.md` | 何をさせる指示か。外部のサイトへの送信や、勝手な公開を指示していないか |
| `scripts/`、`hooks/`、`bin/` | 手元で何が動くか。外へ通信するか、別の実行ファイルを落としてくるか |
| README の「telemetry」「analytics」 | 利用状況を外へ送るか。切り方 |
| 依存するもの | Node.js・Python・Chrome などの版 |
| ライセンス | MIT・Apache-2.0 なら職場でも扱いやすい。「個人利用に限る」なら職場では使わない |
| 偽物の注意 | README に「公式の配布元はここだけ」とあれば、その配布元から入れる |

この号で見た実例:

- impeccable は、既定で設計の選択を外のサーバーへ送る（切り方は 06）
- superpowers は、画面に出すロゴを外から読み込み、そのとき版の番号が伝わる（切り方は 07）
- skills CLI 自身も、匿名の利用状況を送る（切り方は下）

### 2. 入れ方

#### Claude Code

プラグインの形（多くのスキルがこれを用意している）。リポジトリの README に書かれた命令をそのまま使う。形はおおむね次のどちらか:

```
/plugin marketplace add <持ち主>/<リポジトリ>
```

```
/plugin install <プラグイン名>@<マーケットプレイス名>
```

Anthropic の公式のマーケットプレイス（`claude-plugins-official`）に載っているものは、`/plugin install <名前>@claude-plugins-official` で入る（例: 07 の superpowers）。

フォルダを置く形: `~/.claude/skills/<スキル名>/` に、`SKILL.md` のあるフォルダを置く（全部のプロジェクトで効く）。そのプロジェクトだけなら `.claude/skills/<スキル名>/`。

Windows では `~` は利用者のホームフォルダ（`C:\Users\<名前>`）。

#### skills CLI（ほかのエージェント、または Claude Code）

```bash
npx skills add <持ち主>/<リポジトリ>
```

よく使う指定（skills CLI の README のとおり）:

| 指定 | 意味 |
|---|---|
| `-g` / `--global` | 利用者のフォルダに入れる（全部のプロジェクトで効く）。付けなければ今のプロジェクトだけ |
| `-a <名前>` / `--agent <名前>` | 入れる先のエージェント。例 `-a claude-code`、`-a codex`、`-a cursor` |
| `--skill <名前>` | リポジトリの中の1つだけ入れる |

```bash
npx skills list
```

```bash
npx skills update
```

```bash
npx skills remove <スキル名>
```

入る場所: Claude Code はプロジェクトなら `.claude/skills/`、`-g` なら `~/.claude/skills/`。Codex・Cursor・OpenCode・Gemini CLI はプロジェクトなら `.agents/skills/`。

利用状況の送信を切る（skills CLI の README のとおり）: 環境変数 `DISABLE_TELEMETRY=1` か `DO_NOT_TRACK=1`。

```bash
export DISABLE_TELEMETRY=1
```

Windows の PowerShell なら:

```powershell
$env:DISABLE_TELEMETRY = "1"
```

#### claude.ai

設定 → Capabilities → Skills で、スキルのファイル（`.skill` または zip）を上げる（html-artifacts の README の手順）。対応しているスキルは README にそう書いてある（例: 05 の archify は `archify.zip` を上げる形を用意している）。

#### ChatGPT などのチャット

`SKILL.md` の中身を会話の最初に貼り、「以後この指示に従って」と頼む。`SKILL.md` が別のファイル（`references/` など）を読みに行く形のスキルは、その部分が効かない。

### 3. 入れた後

- 利用者に「何を入れたか、どこに入ったか、外へ送るものがあるか、切ったか」を1行ずつ報告する
- 同じ役割のスキル（テストの進め方、レビューの仕方など）を複数入れると、どちらに従うかがぶつかる（推測）。**一度に1つ入れて、効き目を見てから次を入れる**

### 出典（2026-10-17 に読んだもの）

- [vercel-labs/skills の README](https://cdn.jsdelivr.net/gh/vercel-labs/skills@main/README.md)（命令、入る場所、送信の切り方）
- [dogum/html-artifacts](https://github.com/dogum/html-artifacts)（claude.ai への入れ方）
- 05〜08 の各リポジトリの README
