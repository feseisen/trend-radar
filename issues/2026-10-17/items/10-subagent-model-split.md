# 10 サブエージェントごとにモデルを割り当て直す（「写すだけ」か「判断が要る」かで線を引く）

号: 2026-10-17 ／ 判定: **採用**（新顔） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: Claude Code の「サブエージェント」（親の AI が仕事を任せる子の AI）に、役割ごとに **どのモデルを、どの深さ（effort）で使うか** を決め直すやり方。線の引き方は **「結果を決まった形に写すだけか、判断や漏れの無さが要るか」**（[Zenn の記事、株式会社GENDA](https://zenn.dev/genda_jp/articles/haiku-5-5-subagent-roles)、2026-10-08）
- **なぜいま話題か**: 新しい軽いモデルが出たのをきっかけに見直した記事が、Zenn の AI 関連の直近30日の人気で 2位（♥48、10-11 時点のまとめ）
- **記事の結論**（記事の筆者が手元で少ない回数を比べた結果。統計的な評価ではないと明記）:
  - 試験を走らせて結果を写すだけの役 → 軽いモデル、深さは低で十分
  - コードを調べる役 → 答えの質は軽いモデルでも変わらなかったが、時間は同じか長め。**速さを取るか費用を取るか** で決まる
  - 実装する役 → 仕様とのずれの判断や確認があるので、中くらいのモデルに残した
  - 中くらいのモデルの深さを「中」と「高」で比べると、質に差が見えず、「高」は時間とトークンを多く使った
- **取り入れると何が良くなるか**（推測）: 枠や費用の減りが遅くなる。待ち時間が減る役もある
- **手間**: 定義のファイルを数行直すだけ。比べて確かめるなら1〜2時間
- **気をつけること**: 深さを「低」にすると、長い指示の中で調べるのを省いたり途中で止まったりしやすい（記事が引く Prompting ガイドの注意）。「中」でも確かめずに完了と言うことがある。だから **報告の型を決めて、確かめたかどうかを書かせる**

### 対応環境

| | |
|---|---|
| 使える AI | Claude Code（サブエージェントの定義ファイル）。考え方はほかの道具にも使える |
| 要るもの | 特に無い |
| ネット・API キー | 使っている Claude Code の契約のまま。追加は無い |
| 外への送信 | 無い |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者のプロジェクトのサブエージェント（`.claude/agents/*.md`、`~/.claude/agents/*.md`）を見直し、役割ごとにモデルと深さを提案してください。**決める前に、下の「確かめ方」で小さく比べてください。** 比べずに決めた割り当ては「推測」と書いて利用者に渡してください。

### 1. 線の引き方

各サブエージェントの仕事を、次のどちらかに分ける。

| 種類 | 例 | 向くモデル（記事の結論） |
|---|---|---|
| **写すだけ**: 出力から決まった項目を抜き出して返す | 試験・ビルド・命令を走らせて結果を報告する | 軽いモデル、深さ 低 |
| **判断・漏れの無さが要る**: 何を調べるか、どこで止めるかを自分で決める | コードを調べる、実装する、点検する | 中くらい以上。調べる役は速さ優先なら中くらい、費用優先なら軽いモデルも候補 |

**取捨選択がある仕事を軽いモデルに渡さない。**

### 2. 定義ファイルの書き方（Claude Code の公式文書による）

置き場:

| 置き場 | 効く範囲 |
|---|---|
| `.claude/agents/` | そのプロジェクト（Git に入れてチームで共有できる） |
| `~/.claude/agents/` | 利用者の全部のプロジェクト |

公式文書の例（そのまま）:

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

使える項目のうち、この話で使うもの（公式文書の表より）:

| 項目 | 値 |
|---|---|
| `model` | `sonnet`、`opus`、`haiku`、`fable`、モデルの正式な ID（例 `claude-opus-5-5`）、または `inherit`（親と同じ） |
| `effort` | `low`、`medium`、`high`、`xhigh`、`max`（使える段はモデルによる） |
| `tools` | 使ってよい道具。写すだけの役は `Bash` と `Read` 程度に絞る |

**項目名は表のとおりに書く。** 綴りが違う項目は、Claude Code が何も言わずに無視する（公式文書の注意）。

モデルが決まる順（公式文書のとおり）: 呼ぶときに指定したモデル → 定義ファイルの `model` → 環境変数 `CLAUDE_CODE_SUBAGENT_MODEL` → 親の会話のモデル。

### 3. 例（この号で組み立てたもの。記事の割り当てと報告の型を、公式文書の書き方に当てはめた）

`.claude/agents/command-runner.md`:

```markdown
---
name: command-runner
description: Runs a given test, build, or shell command and reports the result in a fixed format. Use for running commands, not for deciding what to change.
tools: Bash, Read
model: haiku
effort: low
---

Run exactly the command you are given. Do not fix anything.
Begin your report with this block, then at most 20 lines of the most relevant output:

Command: <the command>
CWD: <working directory>
Exit code: <number>
Status: PASSED | FAILED | DID NOT RUN | TIMED OUT
Summary: <one line: counts and the first failure>
```

`.claude/agents/code-explore.md`:

```markdown
---
name: code-explore
description: Investigates the codebase to answer a question (where something is, how a flow works, what uses what). Read-only.
tools: Read, Grep, Glob
model: sonnet
effort: medium
---

Answer the question by reading the code. Keep the whole report within about 60 lines, in this order:

Answer: <the answer in one to three lines>
Evidence: <file:line references that support it>
Inferences: <what you concluded but did not see directly, marked as such>
Search log: <patterns and paths you searched>
Not covered: <what you did not look at>
Noticed: <anything important you saw on the way>

Separate facts from inferences. Any claim that something does not exist must list the search patterns you used.
```

`.claude/agents/spec-coder.md`:

```markdown
---
name: spec-coder
description: Implements a change from a written spec and verifies it. Use when the spec is clear.
model: sonnet
effort: high
---

Implement the spec. When the spec and the code disagree, stop and report the conflict instead of guessing.
End with these four sections:

Changed files: <list>
Not done: <what you did not do, and why>
Deviations: <where you departed from the spec, and why>
Verification: <what you ran to check it, and the result>
```

報告の型の項目（Command、CWD、Exit code、Status の4値、Answer〜Noticed の6項目と約60行、Changed files〜Verification の4項目）は記事のとおり。文の言い回しはこちらで書いた。

### 4. 確かめ方（決める前に）

1. その役のふだんの頼みを 3〜5 個選ぶ（調べる役なら「〇〇はどこで決まっているか」「この流れを追って」など）
2. 同じ頼みを、モデルか深さだけ変えて2回ずつ走らせる。定義ファイルは変えず、呼ぶときの指定で切り替える
3. 比べるもの: 答えが正しいか、漏れが無いか、かかった時間、使ったトークン
4. 結果を表にして利用者に渡す。**回数が少ないことを必ず書く**

### 5. 注意

- 深さ「低」は、長い指示の中で調べるのを省いたり、途中で止まったりしやすい。「中」でも確かめずに完了と言うことがある（記事が引く注意）。報告の型に「確かめたこと」の欄を必ず置く
- 定額の契約で使うなら速さが効き、従量の課金で使うなら費用が効く（記事の筆者の整理）。利用者がどちらかを先に聞く
- 記事は、軽いモデルの料金がプロンプトの長さ（10万トークン）を境に上がると書いている。大きなリポジトリを長く調べさせる役では、この境を超えないか見る（料金の数字は記事の値で、公式の料金表では確かめていない）

### 出典（2026-10-17 に読んだもの）

- [Zenn: Haiku 5.5 を機に Sonnet 以下で動かしていたサブエージェントを見直した](https://zenn.dev/genda_jp/articles/haiku-5-5-subagent-roles)（2026-10-08、株式会社GENDA）
- [Claude Code の公式文書: Subagents](https://code.claude.com/docs/en/subagents)（項目の表、置き場、例、モデルが決まる順）
