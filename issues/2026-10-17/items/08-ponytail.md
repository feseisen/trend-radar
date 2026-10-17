# 08 ponytail（AIに「コードを少なく、簡単に」と言い聞かせる）

号: 2026-10-17 ／ 判定: **採用**（定番入り。効果の数字は作者自身の測定） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: AIエージェントが書くコードを減らし、簡単な解き方を選ばせる指示書（[リポジトリ](https://github.com/DietrichGebert/ponytail)、MIT）。書く前に「そもそも要るか」「もうあるか」「標準の機能で足りるか」…と7段の梯子を上から確かめさせる
- **なぜいま話題か**: 10月の GitHub の伸びで +15.3k（集計サイト）。合計 165.3k、フォーク 8.9k（2026-10-17 の現物）
- **作者の測定**（作者自身による。第三者の再現は示されていない）: 39 の課題を同じモデルで各5回、スキルあり・なしで比べて、コード −53%、時間 −41%、費用 −26%、トークン −45%。危ない処理に試験が付いた割合 98%（なしは 68%）
- **取り入れると何が良くなるか**（推測）: AIが作る小さな道具やスクリプトが短くなり、読んで確かめやすくなる
- **手間**: Claude Code なら2行。ほかの AI でも、指示文（`AGENTS.md`）を写すだけで使える
- **気をつけること**: 数字は作者の測定で、モデルは1つ、課題の選び方は書かれていない。README は「公式の配布元はリポジトリと npm だけ、`.exe` や `.dll` は配らない」と偽物に注意している

### 対応環境

| | |
|---|---|
| 使える AI | 命令まで使えるのは Claude Code、Codex、Devin CLI、OpenCode、Gemini、Antigravity CLI、pi、Hermes Agent、Qoder、Grok Build。指示文だけなら Cursor、Windsurf、Cline、Copilot、Kiro など。ChatGPT などには中身を貼る |
| 要るもの | 特に無い（指示文だけなら）。Claude Code の画面の上の帯の表示は Claude Code 2.1.290 以上 |
| ネット | 入れるときだけ |
| API キー | 要らない |
| ライセンス | MIT |
| 外への送信 | README に記述なし。帯の記録は手元の `~/.config/ponytail/band/` に置かれる |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「AIの書くコードを短くしたい」「余計なものを作らせたくない」と言ったら、ponytail を入れてください（04 の点検をしてから）。入れられない環境（チャットだけ）なら、下の「指示の中身」を会話の最初に置いて、以後それに従ってください。

### 入れる

Claude Code（README のとおり）:

```
/plugin marketplace add DietrichGebert/ponytail
```

```
/plugin install ponytail@ponytail
```

Codex:

```bash
codex plugin marketplace add DietrichGebert/ponytail
codex plugin add ponytail@ponytail
```

Codex では、その後 `/hooks` を開いて2つの hook を信頼し、新しいスレッドを始める（README のとおり）。

ほかの AI: リポジトリの `AGENTS.md` をプロジェクトに写すか、`skills/ponytail/SKILL.md` をスキルとして入れる。

### 指示の中身（README の要約。正確な文言はリポジトリの `SKILL.md` を読む）

1. まず問題を理解し、変更が触るコードを読む
2. 書く前に、次の梯子を上から確かめ、**最初に当てはまった段で止まる**
   1. これは要るのか
   2. もうコードの中にあるか
   3. 標準ライブラリで足りるか
   4. 動かす環境の標準の機能で足りるか
   5. 入っている依存で足りるか
   6. 1行で書けるか
   7. ここまで来て初めて、動く最小のものを書く
3. 信頼の境目の入力の検査、データを失う処理、セキュリティ、アクセシビリティは **削らない**
4. 分岐・繰り返し・解析・お金・セキュリティを含む処理には、小さな試験を1つ付ける
5. 返事の最後に、省いたこと・確かめていないこと・危うい点を書く
6. わざと手を抜いた所には `shortcut:` という注記を付ける

### 命令（Claude Code など）

| 命令 | すること |
|---|---|
| `/ponytail [lite\|full\|ultra\|off]` | 強さを変える |
| `/ponytail-review` | 変更を点検する |
| `/ponytail-audit` | まとめて点検する |
| `/ponytail-debt` | 手を抜いた所（`shortcut:`）を数える |
| `/ponytail-gain` | 効き目を見る |
| `/ponytail-help` | 説明 |

### 守ること

- 「削らない」4つ（入力の検査・データを失う処理・セキュリティ・アクセシビリティ）を、短くするために削らない
- 利用者が書いた既存のコードを、頼まれていないのに大きく書き換えない

### 出典（2026-10-17 に読んだもの）

- [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)（星 165.3k、フォーク 8.9k、コミット 355、MIT）
- 作者の測定の詳しい条件はリポジトリの `benchmarks/results/2026-10-07-agentic.md` にあるとされる（この号では読んでいない）
