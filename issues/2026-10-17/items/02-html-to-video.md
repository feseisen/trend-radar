# 02 HTML から動画を作る（HyperFrames）

号: 2026-10-17 ／ 判定: **採用** ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: 動画を「動画編集ソフト」でも「動画生成AI」でもなく、**HTML とアニメーションのコードで書いて MP4 にする** やり方。HyperFrames はその道具で、動画の会社 HeyGen が公開している（[リポジトリ](https://github.com/heygen-com/hyperframes)）
- **なぜいま話題か**: 「AIにコードで動画を作らせる」作例が X で伸びている（01 と同じ流れ）。HyperFrames は「エージェントが書く前提」を名乗っており、10月に GitHub で星が約 1 万増えた（集計サイトの値）
- **取り入れると何が良くなるか**（推測）: 文字が崩れない、秒やコマの単位で直せる、同じ入力から同じ動画ができる。説明動画・告知の短い動画・数字が動くグラフなどに向く
- **手間**: Node.js 22 以上と FFmpeg を入れれば、試すのは数分。Claude Code ならプラグインで入る
- **気をつけること**: よく似た道具の Remotion は、**従業員4人以上の営利企業では有料のライセンスが要る**（[Remotion の LICENSE](https://github.com/remotion-dev/remotion/blob/main/LICENSE.md)）。職場で使うなら HyperFrames（Apache-2.0）のほうが扱いやすい。完成の姿を先に決めないと納得できない、は 01 と同じ

### 対応環境

| | |
|---|---|
| 使える AI | Claude Code（プラグインあり）。ほかのエージェントは skills CLI で入る（04 を参照） |
| OS | 書かれていない。Git LFS の入れ方は macOS・Ubuntu・Windows の分が載っている |
| 要るもの | Node.js 22 以上、FFmpeg。描画は画面なしの Chrome を使う（別途 Chrome が要るかは書かれていない） |
| ネット | 入れるときは要る。例ではアニメーションの道具（GSAP）を CDN から読む |
| API キー | 動画を作るだけなら要らない（README に記述なし）。読み上げ・音楽・画像を生成させる機能には何かの提供元が要るらしいが、書かれていない |
| ライセンス | Apache-2.0 |
| 外への送信 | 任意で HeyGen のクラウドで描画できる（使わなければ手元で完結すると読める） |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「〇〇の短い動画を作って」と頼んだら、HyperFrames で HTML の動画を作り、MP4 に書き出してください。
**書き方の細かい決まりは、このメモではなく HyperFrames に付いてくるスキルに従ってください。** このメモは入れ方と進め方だけを書いています（決まりは版で変わるため）。

### 1. 前提を確かめる（作業の前に必ず）

```bash
node -v
ffmpeg -version
```

- `node -v` が `v22` 以上でなければ止まり、利用者に Node.js 22 以上を入れてもらう
- `ffmpeg` が見つからなければ止まり、利用者に FFmpeg を入れてもらう（入れ方は端末の決まりに従う。職場の端末では管理者に頼む必要があるかもしれない）

### 2. 入れる

Claude Code なら（README のとおり）:

```
claude plugin marketplace add heygen-com/hyperframes
```

```
claude plugin install hyperframes@hyperframes
```

ほかのエージェントなら:

```bash
npx skills add heygen-com/hyperframes
```

入れたら、Claude Code では `/hyperframes:hyperframes`、スキルとして入れた場合は `/hyperframes` で呼べる。

### 3. 作る

```bash
npx hyperframes init my-video
cd my-video
npx hyperframes preview
```

- 動画は HTML で書く。根の要素に作品の id と大きさ（例 1920×1080）を持たせ、映像・文字・音の各要素に「何秒から・何秒間・何番目の段」を属性で書く。動きは GSAP などの止めてあるタイムラインに書いて登録する（README の例の形。正確な属性名と登録の仕方は、入れたスキルと `init` が作る雛形に従う）
- アニメーションは GSAP のほか、Lottie、Anime.js、CSS、Web Animations API、Three.js も使える（README の対応表）
- 例の部品は `npx hyperframes add data-chart` のように足せる

### 4. 書き出す

```bash
npx hyperframes render
```

画面なしの Chrome で1コマずつ描き、FFmpeg で MP4 にする（README の説明）。

### 5. 進め方（品質のために）

1. 利用者と **秒ごとの流れ** を先に決める（「0〜2秒: 題が浮かぶ、2〜6秒: 数字が伸びる…」）。決まらないまま作らない
2. 最初は短く（5〜10秒）作って `preview` で見てもらう
3. 直すのは一度に1点ずつ（「題の出る速さだけ遅く」）
4. 長くするのは、短いものが通ってから

### やってはいけないこと

- 利用者に断らずに、クラウドでの描画（HeyGen のクラウド、AWS Lambda）を使わない。素材が外に出る
- 素材（動画・音楽・画像）の権利が分からないものを使わない
- Remotion に切り替えるなら、先に利用者に「従業員4人以上の営利企業では有料のライセンスが要る」と伝える

### うまくいかないとき

| 症状 | 見るところ |
|---|---|
| `render` が途中で止まる | FFmpeg の有無、Node.js の版 |
| リポジトリを `git clone` すると大きなファイルで失敗する | README の手順どおり `GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/heygen-com/hyperframes.git` を使う（clone しないで `npx` だけで使うなら不要） |
| 文字の動きがぎこちない | イージング（動きの緩急）を指定しているか。秒の割り振りが短すぎないか |

### 出典（2026-10-17 に読んだもの）

- [heygen-com/hyperframes](https://github.com/heygen-com/hyperframes)（星 65.5k、フォーク 5.9k、Apache-2.0）
- [Remotion の LICENSE](https://github.com/remotion-dev/remotion/blob/main/LICENSE.md): 無料で使えるのは個人、従業員3人までの営利企業、非営利団体、評価目的。それ以外は Company License
- [notdesignschool](https://notdesignschool.jp/story/claude-opus-5-5-for-designers): デザイナーの界隈でのコードで作る動画の作例と、Remotion のライセンスの注意
