# 07 開発の進め方のスキル集（定番3つ: superpowers、mattpocock/skills、addyosmani/agent-skills）

号: 2026-10-17 ／ 判定: **採用**（定番入り。新しくはないが、まだ入れていないなら最初に入れる価値がある） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: AIエージェントに「仕事の進め方」を教えるスキル集。いきなりコードを書かせず、意図を確かめる → 計画を立てる → 小さく作って試す → 点検する、の順に動かせる
  - **superpowers**（[リポジトリ](https://github.com/obra/superpowers)、MIT、作者 Jesse Vincent）: 進め方そのもの（方法論）。15 のスキルが一続きの流れになっている。いちばん重い
  - **mattpocock/skills**（[リポジトリ](https://github.com/mattpocock/skills)、MIT、作者 Matt Pocock）: 27 のスキルから、要るものを選んで使う形。開発だけでなく、**計画の穴を質問で詰める**（grill-me）、**会話を別の AI への引き継ぎ書にする**（handoff）、**分かりにくい返事を言い直させる**（wait-what）など、仕事一般に効くものがある
  - **addyosmani/agent-skills**（[リポジトリ](https://github.com/addyosmani/agent-skills)、MIT、作者 Addy Osmani）: 定義・計画・作る・確かめる・点検・出す、の段階ごとの 25 のスキル。各スキルに「AIが手順を飛ばすときの言い訳と、その反論」の表が付いている
- **なぜいま話題か**: どれも合計の星が大きい定番（約 30 万、29.4 万、10.6 万。2026-10-17 の現物）。10月の伸びは mattpocock が +19.9k、ほかは +6〜7k（集計サイト）。**最新の流行りではなく、定着したもの**
- **取り入れると何が良くなるか**（推測）: 「AIが勝手に作り始めて、意図と違うものができる」が減る
- **手間**: 入れるのは1〜2行
- **気をつけること**: 3つを同時に入れない（役割がぶつかる。推測）。1つ選び、慣れてから足す。superpowers は画面に出すロゴを外から読み込む（切れる）

### どれを選ぶか（推測による目安）

| こうしたいなら | 選ぶ |
|---|---|
| 進め方ごと任せたい。大きめの開発 | superpowers |
| 要るものだけ足したい。開発以外の仕事にも使いたい | mattpocock/skills |
| 点検の観点（セキュリティ・速さ・文書）を厚くしたい | addyosmani/agent-skills |

### 対応環境

| | superpowers | mattpocock/skills | addyosmani/agent-skills |
|---|---|---|---|
| 使える AI | Claude Code、Codex、Cursor、Gemini CLI、GitHub Copilot CLI ほか多数 | Claude Code、Codex、GitHub Copilot、Gemini CLI ほか | Claude Code（推奨）、Cursor、Gemini CLI、Codex ほか。指示文を受け付ける AI なら何でも、と README |
| 要るもの | 使う道具のプラグインの仕組み | 同左。`npx` を使うなら Node.js | 同左 |
| ネット | 入れるとき。画面のロゴの読み込み（切れる） | 入れるとき | 入れるとき |
| API キー | 要らない | 要らない | 要らない |
| ライセンス | MIT | MIT | MIT |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「AIの開発の進め方を整えたい」と言ったら、上の「どれを選ぶか」で1つを勧め、04 の点検をしてから入れてください。**2つ目を入れるのは、1つ目で小さな仕事を何回か回して、効き目が分かってからにするよう利用者に勧めてください**（同じ役割のスキルがぶつかるのを避けるため。推測）。

### superpowers

入れる（Claude Code、Anthropic の公式のマーケットプレイスから。README のとおり）:

```
/plugin install superpowers@claude-plugins-official
```

作者のマーケットプレイスから入れるなら:

```
/plugin marketplace add obra/superpowers-marketplace
```

```
/plugin install superpowers@superpowers-marketplace
```

Codex CLI: `/plugins` を開き、`superpowers` を探して「Install Plugin」。

流れ（README の記載）:

1. **brainstorming**: 作る前に質問して意図を確かめ、理解を言い直して訂正をもらう。小さな変更は会話で、大きなものは設計書を書く
2. **using-git-worktrees**: 設計が通ったら、別の作業場（worktree）を作り、試験が通る状態から始める
3. **writing-plans**: 作業を小さく分ける（ファイルの場所・コード・確かめ方つき）
4. **subagent-driven-development** か **executing-plans**: 作業ごとに新しいサブエージェントで作って点検する、または一続きで作って最後に点検する
5. **test-driven-development**: 失敗する試験 → 通す → 整える
6. **requesting-code-review**: 計画と照らして点検。重大な問題は先へ進ませない
7. **finishing-a-development-branch**: 試験を確かめ、まとめ方（取り込む・PR・残す・捨てる）を選ぶ

外への読み込みを切る（README のとおり）: 環境変数 `SUPERPOWERS_DISABLE_TELEMETRY` を設定する。Claude Code の `DISABLE_TELEMETRY` と `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` にも従う、とある。読み込むのは版の番号の入ったロゴで、作業や指示の中身は送らない、と README は書いている。

注意: 新しいスキルの持ち込みは原則受け付けていない（README の記載）。

### mattpocock/skills

入れる（Claude Code。README のとおり）:

```
claude plugin install mattpocock-skills@claude-plugins-official
```

ほかのエージェント:

```bash
npx skills@latest add mattpocock/skills
```

開発以外でも効くもの（README の説明を短くしたもの）:

| スキル | すること |
|---|---|
| `grill-me` | 計画について、設計の問いが片付くまで質問し続ける |
| `handoff` | 会話を、別の AI が続きをやれる文書にまとめる |
| `wait-what` | 伝わらなかった返事を、平易に言い直す |
| `to-questionnaire` | 決まっていないことを、別の人に答えてもらう質問票にする |
| `research` | 一次資料から調べ、出典つきの Markdown に残す |
| `writing-for-agents` | スキルや AGENTS.md / CLAUDE.md など、AI 向けの文書の書き方 |

開発で中心になるもの: `ask-matt`（どのスキルを使うかの案内）、`to-spec`、`to-tickets`、`implement`、`tdd`、`diagnosing-bugs`、`code-review`。
`setup-matt-pocock-skills` はリポジトリごとに1回動かす設定（イシューの置き場や文書の配置を決める）。

### addyosmani/agent-skills

入れる（README のとおり）:

```bash
npx skills add addyosmani/agent-skills
```

Claude Code のプラグインとして:

```
/plugin marketplace add addyosmani/agent-skills
```

```
/plugin install agent-skills@addy-agent-skills
```

1つだけ入れる例:

```bash
npx skills add addyosmani/agent-skills --skill interview-me
```

**注意（README の記載）**: 1つだけ入れると `skills/<名前>/` しか写らず、共有の点検表（`references/`）が付いてこない。点検表まで効かせたいなら、まとめて入れる。

段階ごとの主なスキル: 定義（`interview-me`、`spec-driven-development`）、計画（`planning-and-task-breakdown`）、作る（`incremental-implementation`、`test-driven-development`、`context-engineering`）、確かめる（`browser-testing-with-devtools`）、点検（`code-review-and-quality`、`security-and-hardening`）、出す（`git-workflow-and-versioning`、`shipping-and-launch`）。

### どれでも守ること

- 入れたら、利用者の小さな仕事を1つ、そのスキルの流れで最後までやってみせる
- 流れが重すぎると利用者が感じたら、軽い形（会話だけで済ませる、点検を最後の1回にする）を提案する

### 出典（2026-10-17 に読んだもの）

- [obra/superpowers](https://github.com/obra/superpowers)（星 約 299.8k、フォーク 26.8k、イシュー 208、PR 150、MIT）
- [mattpocock/skills](https://github.com/mattpocock/skills)（星 294.1k、フォーク 24.7k、MIT）
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)（星 105.9k、フォーク 11.1k、MIT）
