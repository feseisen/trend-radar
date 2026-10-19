# 04 スキルの入れ方（05〜08 の土台）

号: 2026-10-17 ／ 判定: **採用**（定番入り。05〜08 を入れるための共通の土台） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: AIエージェントに足す「スキル」は、**`SKILL.md` という指示書の入ったフォルダ** で、ツールが違っても同じ形になっている。この号で取り上げたスキル（05〜08）は、どれもこの形で配布されている
- **なぜいま話題か**: 10月に GitHub で伸びたリポジトリの上位に、スキル集が並んでいる（集計サイトの 10月の表で、AI 関係 27 件のうち、スキルそのもの、またはスキルを同梱するものが 10 件）。インストール用のツールも共通化が進み、`npx skills` という1つのコマンドで 70 以上のエージェントに入れられる（[skills CLI の README](https://cdn.jsdelivr.net/gh/vercel-labs/skills@main/README.md)）
- **取り入れると何が良くなるか**（推測）: 一度覚えれば、新しいスキルが出るたびに同じ手順で試せる
- **手間**: 1つ入れるのに1〜2分
- **気をつけること**: スキルは「指示書」だけでなく、**手元で動くスクリプトや、外へ送る通信を含むことがある**。入れる前に中身を見る（下のチェック表）

### 入れ方は使うツールで4通り

| 使うツール | 入れ方 | 備考 |
|---|---|---|
| Claude Code | プラグインとして入れる、またはフォルダを置く | この入れ方を用意しているスキルがいちばん多い |
| ほかのエージェント（Codex、Cursor など） | `npx skills add` | Node.js が要る |
| claude.ai | 「カスタマイズ → スキル」で zip をアップロードする（README によっては `.skill` も挙げている） | 対応しているスキルだけ |
| ChatGPT などのチャット | `SKILL.md` の中身を貼る | 複数ファイルのスキルは一部しか効かない |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「このスキルを入れて」と頼んだら、下の手順で **チェックしてから** 入れてください。チェックで引っかかったものは、入れる前に利用者に報告して判断を仰いでください。

### スキルとは

- スキルは、`SKILL.md` を入口に持つフォルダ。`SKILL.md` の先頭には YAML で `name` と `description` が書かれている（skills CLI の README の説明）
- 補助の文書（`references/` など）やスクリプト（`scripts/`）を同じフォルダに持つことがある

### 1. 入れる前のチェック（必ず）

リポジトリを開き、次を見て、利用者に1つずつ報告する。

| 見るところ | 何を探すか |
|---|---|
| `SKILL.md` | 何をさせる指示か。外部のサイトへの送信や、勝手な公開を指示していないか |
| `scripts/`、`hooks/`、`bin/` | 手元で何が動くか。外へ通信するか、別の実行ファイルをダウンロードしてくるか |
| README の「telemetry」「analytics」 | 利用状況を外へ送るか。送るなら、オフにする方法 |
| 依存するもの | Node.js・Python・Chrome などのバージョン |
| ライセンス | MIT・Apache-2.0 なら職場でも扱いやすい。「個人利用に限る」なら職場では使わない |
| 偽物の注意 | README に「公式の配布元はここだけ」とあれば、その配布元から入れる |

この号で見た実例:

- impeccable は既定で、画面づくりを始めるときに選んだ選択肢を外のサーバーへ送る（オフにする方法は 06）
- superpowers は、画面に出すロゴを外から読み込み、そのときバージョン番号が伝わる（オフにする方法は 07）
- skills CLI 自身も、匿名の利用状況を送る（オフにする方法は下）

### 2. 入れ方

#### Claude Code

プラグインの形（多くのスキルがこれを用意している）。リポジトリの README に書かれたコマンドをそのまま使う。形はおおむね次のどちらか:

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

よく使うオプション（skills CLI の README のとおり）:

| オプション | 意味 |
|---|---|
| `-g` / `--global` | ユーザー単位で入れる（全部のプロジェクトで効く）。付けなければ今のプロジェクトだけ |
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

1. コード実行（「コード実行とファイル作成」）を有効にする。個人向けのプラン（Free・Pro・Max）なら、自分で「設定 → 機能」からオンにする。Team・Enterprise プランでは、組織のオーナーが組織の設定で決める（Team は、スキルが最初から有効になっている）。使えないときは組織のオーナーに確かめる
2. 「カスタマイズ → スキル」を開き、「+」→「+ スキルを作成」→「スキルをアップロード」と進む
3. スキルの zip ファイルをアップロードする（フォルダしか無いときは、フォルダを zip にまとめる）

画面の名前とプランごとの違いは、[公式ヘルプ（日本語版）](https://support.claude.com/ja/articles/12512180)（2026-10-18 に読んだ）に合わせた。html-artifacts の README は英語の画面名（Settings → Capabilities → Skills）で書いていて、`.skill` のファイルも挙げているが、公式ヘルプが挙げているのは zip だけ。

claude.ai に入れられる形を用意しているスキルは、README にそう書いてある（例: 05 の archify は `archify.zip` をアップロードする形を用意している）。

#### ChatGPT などのチャット

`SKILL.md` の中身を会話の最初に貼り、「以後この指示に従って」と頼む。`SKILL.md` が別のファイル（`references/` など）を読みに行く形のスキルは、その部分が効かない。

### 3. 入れた後

- 利用者に「何を入れたか、どこに入ったか、外へ送るものがあるか、切ったか」を1行ずつ報告する
- 同じ役割のスキル（テストの進め方、レビューの仕方など）を複数入れると、指示がぶつかり、どちらに従うかが定まらなくなる（推測）。**一度に1つ入れて、効き目を見てから次を入れる**

### 出典（2026-10-17 に読んだもの）

- [vercel-labs/skills の README](https://cdn.jsdelivr.net/gh/vercel-labs/skills@main/README.md)（コマンド、入る場所、送信の切り方）
- [dogum/html-artifacts](https://github.com/dogum/html-artifacts)（claude.ai への入れ方）
- 05〜08 の各リポジトリの README
