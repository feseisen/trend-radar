# 02 HTML から動画を作る（HyperFrames）

号: 2026-10-17 ／ 判定: **採用** ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: 動画を「動画編集ソフト」でも「動画生成AI」でもなく、**HTML とアニメーションのコードで書いて MP4 にする** やり方。HyperFrames はそのためのツールで、動画の会社 HeyGen が公開している（[リポジトリ](https://github.com/heygen-com/hyperframes)）
- **なぜいま話題か**: 「AIにコードで動画を作らせる」作例が X で伸びている（01 と同じ流れ）。HyperFrames は「エージェントが書く前提」と明記しており、10月に GitHub でスターが約 1 万増えた（集計サイトの値）
- **取り入れると何が良くなるか**（推測）: 文字が崩れない、秒やコマの単位で直せる、同じ入力から同じ動画ができる。説明動画・告知の短い動画・数字が動くグラフなどに向く
- **手間**: Node.js 22 以上と FFmpeg を入れれば、試すのは数分。Claude Code ならプラグインで入る
- **気をつけること**: よく似たツールの Remotion は、**従業員4人以上の営利企業では有料のライセンスが要る**（[Remotion の LICENSE](https://github.com/remotion-dev/remotion/blob/main/LICENSE.md)）。職場で使うなら HyperFrames（Apache-2.0）のほうが扱いやすい。完成イメージを先に決めないと、何度直しても納得のいくものにならない（01 と同じ注意）

### 対応環境

| | |
|---|---|
| 使える AI | Claude Code（プラグインあり）。ほかのエージェントは skills CLI で入る（04 を参照） |
| OS | README に対応 OS の記載は無い |
| 要るもの | Node.js 22 以上、FFmpeg。動画の書き出し（レンダリング）には、画面を出さずに裏で動く Chrome を使う（別途 Chrome が要るかは書かれていない） |
| ネット | 入れるときは要る。例ではアニメーションのライブラリ（GSAP）を CDN（ライブラリなどを配っている外部のサーバー。ここでは jsDelivr）から読み込む |
| API キー | README に API キーの記述は無い（動画を作るだけなら要らないと読める）。読み上げ・音楽・画像を生成する機能には、生成に使うモデルのサービスが別に要りそうだが、どれが要るかは書かれていない |
| ライセンス | Apache-2.0 |
| 外への送信 | 任意で HeyGen のクラウドでレンダリングできる（使わなければ手元で完結すると読める） |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。

### あなた（AI）への依頼

利用者が「〇〇の短い動画を作って」と頼んだら、HyperFrames で HTML の動画を作り、MP4 に書き出してください。
**書き方の細かい決まりは、このメモではなく HyperFrames に付いてくるスキルに従ってください。** このメモは入れ方と進め方だけを書いています（決まりはバージョンで変わるため）。

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

- 動画は HTML で書く。ルート要素に作品の id とサイズ（例 1920×1080）を持たせ、映像・文字・音の各要素に「何秒から・何秒間・何番目のトラック（重ねる層）」を属性で書く。動きは GSAP などの、一時停止した状態のタイムラインに書いて登録する（README の例の形。正確な属性名と登録の仕方は、入れたスキルと `init` が作る雛形に従う）
- アニメーションは GSAP のほか、Lottie、Anime.js、CSS、Web Animations API、Three.js も使える（README の対応表）
- 用意された部品は `npx hyperframes add data-chart` のように追加できる

### 4. 書き出す

```bash
npx hyperframes render
```

ヘッドレスの Chrome で1コマずつ描画し、FFmpeg で MP4 にする（README の説明）。

### 5. 進め方（品質のために）

1. 利用者と **秒ごとの流れ** を先に決める（「0〜2秒: タイトルが浮かぶ、2〜6秒: 数字が伸びる…」）。決まらないまま作らない
2. 最初は短く（5〜10秒）作って `preview` で見てもらう
3. 直すのは一度に1点ずつ（「タイトルの出る速さだけ遅く」）
4. 長くするのは、短いものが通ってから

### やってはいけないこと

- 利用者に断らずに、クラウドでのレンダリング（HeyGen のクラウド、AWS Lambda）を使わない。素材が外に出る
- 素材（動画・音楽・画像）の権利が分からないものを使わない
- Remotion に切り替えるなら、先に利用者に「従業員4人以上の営利企業では有料のライセンスが要る」と伝える

### うまくいかないとき

| 症状 | 見るところ |
|---|---|
| `render` が途中で止まる | FFmpeg の有無、Node.js のバージョン |
| リポジトリを `git clone` すると大きなファイルで失敗する | README の手順どおり `GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/heygen-com/hyperframes.git` を使う（clone しないで `npx` だけで使うなら不要） |
| 文字の動きがぎこちない | イージング（動きの緩急）を指定しているか。秒の割り振りが短すぎないか |

### 出典（2026-10-17 に読んだもの）

- [heygen-com/hyperframes](https://github.com/heygen-com/hyperframes)（スター 65.5k、フォーク 5.9k、Apache-2.0）
- [Remotion の LICENSE](https://github.com/remotion-dev/remotion/blob/main/LICENSE.md): 無料で使えるのは個人、従業員3人までの営利企業、非営利団体、評価目的。それ以外は Company License
- [notdesignschool](https://notdesignschool.jp/story/claude-opus-5-5-for-designers): デザイナーの界隈での、コードで作る動画の作例と、Remotion のライセンスの注意
