# 01 Three.js の1枚もの（AIに3Dのシーンを書かせる）

号: 2026-10-17 ／ 判定: **採用**（新顔） ／ 確かめた日: 2026-10-17

## 人向けの概要

- **何か**: AIに「Three.js（ブラウザで3Dを描く JavaScript のライブラリ）を使った HTML を1枚」書かせて、3Dのシーン・小さなゲーム・動く説明図を作るやり方
- **なぜいま話題か**: 2026-09-22 に新しいモデルが出てから、X で作例が次々に伸びた。ダークソウル風のアクション（表示50万回超）、日本の風景を舟で進む Three.js のシーンなど（[WEEL の記事](https://weel.co.jp/media/innovator/claude-opus-5-5-game/)）。作例とプロンプトを集めたリポジトリには 3D のシーンが14本ある（[awesome-opus5-5-videos](https://github.com/yihui-dev/awesome-opus5-5-videos)）
- **取り入れると何が良くなるか**（推測）: 説明しにくい仕組み・配置・流れを、触って回せる形で見せられる。資料の1枚に動く図を足せる
- **手間**: AIに頼むだけなら数分。ネットにつながらない端末で開ける形にするなら、Node.js があれば追加で数分
- **気をつけること**: 「すごいものを」とだけ頼むと、10回やり直しても納得できないことがある（[Zenn の体験記](https://zenn.dev/actbe_tech/articles/4b8a5d9252fbbc)）。**完成イメージを先に言葉にしてから頼む**のが肝心
- **この号で確かめたこと**: 下の2つの方式（A: Three.js をネットから読み込む／B: 全部を HTML 1枚に埋め込む）のどちらでも、ファイルをダブルクリックで開いた状態（`file://`）で3Dが表示されることを、画面を出さずに動かした Chromium 141（Linux）で確かめた。Windows と、実際の CDN（ライブラリなどを配っている外部のサーバー。ここでは jsDelivr）では試していない

### 対応環境

| | |
|---|---|
| 使える AI | どれでもよい（Claude、ChatGPT など。コードを書ければ足りる） |
| 開く場所 | Chrome・Edge などのブラウザ。WebGL が使えること |
| ネット | 方式A は要る（CDN から Three.js を読み込む）。方式B は要らない |
| 要るもの | 方式B だけ Node.js（作る側の端末に） |
| API キー | 要らない |
| ライセンス | Three.js は MIT |
| 外への送信 | 方式A は CDN へ1回読みに行く。それ以外は無い |

---

## 生成AI向けの詳細

> ここから下を、そのまま AI に渡してください。AI はこれを読んで、利用者の代わりに作業します。

### あなた（AI）への依頼

利用者が「Three.js で〇〇を作って」と頼んだら、**HTML ファイル1枚** で動く3Dのシーンを作ってください。
作る前に、下の「最初に決めること」を利用者と決めます。決まっていないまま作り始めないでください（理由: 完成イメージがあいまいなまま作ると、何度直しても納得されない。実例あり）。

### 最初に決めること（利用者に一度にまとめて聞く）

1. **何を見せたいか**（一文）: 例「倉庫の中で荷物がどう流れるか」
2. **見る人が何をできるか**: 眺めるだけ／ドラッグで回す／クリックで部品の説明が出る
3. **画面の流れ**（動きがあるなら）: 0〜3秒に何、3〜8秒に何、のように秒で
4. **色と雰囲気**: 背景色、主な色2つまで、明るい／暗い
5. **どこで開くか**: ネットにつながる端末か、つながらない（または外部のサイトがブロックされている）端末か → 方式A か方式B かが決まる

利用者が答えに詰まったら、あなたが案を1つ出して「これでよいか」だけを聞いてください。

### 方式A: CDN から読み込む（ネットにつながる端末向け）

- Three.js のバージョンは **固定** する。2026-10-17 時点の最新は `0.186.1`（npm レジストリの情報）。バージョンを書かない URL は使わない（後で中身が変わって壊れるため）
- 公式の導入手順（[threejs.org の Installation](https://threejs.org/manual/en/installation.html)）の import map の形に従う:

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.186.1/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.186.1/examples/jsm/"
  }
}
</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
// ここに場面のコード
</script>
```

- import map は `<head>` の中、スタイルの後に置く（公式の手順のとおり）
- 公式の手順は「HTML をダブルクリックで開くと動かない機能がある。ローカルのサーバーを立てる」と注意している。**この号で試した範囲では**、CDN から読み込むコードは `file://` で開いても描けた（CDN 側が `Access-Control-Allow-Origin: *` を返す前提）。**動かないのは、画像や3Dモデルのファイルを読み込む場合** と見ている（推測）。だから:
  - **画像・3Dモデルのファイルを使わない。** 形はコードで作る（`BoxGeometry`、`CylinderGeometry`、`ExtrudeGeometry` など）
  - どうしてもファイルが要るなら、利用者に `npx serve .` でサーバーを立てて `http://localhost:3000` で開いてもらう（公式の手順）

### 方式B: ネット無しで開ける1枚（外部のサイトがブロックされている端末向け）

作る側の端末に Node.js がある場合の手順。**この号で実際に通した手順** です。

```bash
mkdir scene && cd scene
npm init -y
npm install three@0.186.1 esbuild
```

`scene.js` に、方式A の `<script type="module">` の中身と同じコード（`import * as THREE from 'three';` で始まる）を書く。

```bash
npx esbuild scene.js --bundle --format=iife --minify --outfile=bundle.js
```

`bundle.js` の中身を、HTML の `<script>…</script>` の中にそのまま埋め込む（`type="module"` は付けない）。
埋め込むときは、中身に `</script` という文字列があれば `<\/script` に置き換える。
できた HTML は約 0.55MB（OrbitControls を含めた場合の実測）で、ネット無しでダブルクリックで開ける。

Node.js が無い端末しか無いなら、方式B は作れない。その場合は利用者にそう伝え、方式A にする。

### シーンのコードで守ること

- 描画先の大きさは画面に合わせ、`resize` で追従する。`renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))` で重くなりすぎないようにする
- 動きは `renderer.setAnimationLoop(...)` で回す
- `prefers-reduced-motion` が有効なら、自動の回転や動きを止める
- WebGL が使えないブラウザでは、真っ黒の画面にせず、文字で「この端末では3Dを表示できません」と出す
- 操作の説明（ドラッグで回せる、など）を画面の隅に小さく出す
- 文字（ラベル）が要るなら、3Dの中に描かず、HTML の要素を上に重ねる（文字がにじまず、日本語も崩れない）

### できたかどうかの確かめ方

- 利用者に開いてもらう前に、あなたが使えるツールがあれば自分で確かめる:
  - ブラウザを操作できるツール（Playwright など）があるなら、ページを開いてエラーが出ていないこと、画面の中央のピクセルが背景色と違うこと（何かが描かれていること）を見る
  - ツールが無ければ、コードを読み直して、import の名前とバージョン、閉じ忘れたタグが無いかを確かめる
- 利用者には「開いたら何が見えるはずか」を一文で伝える（例「オレンジの箱がゆっくり回り、ドラッグで向きを変えられる」）

### うまくいかないとき

| 症状 | 見るところ |
|---|---|
| 真っ白・真っ黒 | ブラウザの開発者ツールのコンソールのエラー。CDN がブロックされているなら方式B に切り替える |
| 「Failed to resolve module specifier」 | import map が `<script type="module">` より前に無い、または名前の綴りが違う |
| 画像やモデルが出ない | `file://` で開いている。ファイルを使わずコードで形を作るか、サーバーを立てる |
| 重い・カクつく | オブジェクトの数、影（`castShadow`）、ピクセル比を減らす |

### 参考にしてよい作例

- [awesome-opus5-5-videos](https://github.com/yihui-dev/awesome-opus5-5-videos): 作例とそのプロンプトの一覧（2026-10-12 更新、プロンプト 703 本、うち 3D のシーン 14 本）。各作例は作り手の X の投稿へのリンクと、別のサイトでの作り直しへのリンクを持っている（そのサイトの宣伝を兼ねていると読める）
- 作り手が使っている技術として Canvas、SVG、GSAP、Three.js、GLSL、WebGL、Web Audio が挙がっている

### 出典（2026-10-17 に読んだもの）

- [three.js manual: Installation](https://threejs.org/manual/en/installation.html)
- npm レジストリの情報: `three` の最新は `0.186.1`
- [WEEL: Claude Opus 5.5でゲームは作れる？](https://weel.co.jp/media/innovator/claude-opus-5-5-game/)
- [Zenn: Opus 5.5でモーショングラフィックを作って絶望した話](https://zenn.dev/actbe_tech/articles/4b8a5d9252fbbc)（2026-10-09）
- [notdesignschool: デザイナー向けモーション制作](https://notdesignschool.jp/story/claude-opus-5-5-for-designers)（コードで1コマずつ描く利点: 文字が崩れない、秒・コマ単位で直せる）
- 手元での検証: Chromium 141（Linux、ヘッドレス、SwiftShader）で、方式A（CORS を許す手元のサーバーから読み込む形で代用）と方式B の両方を `file://` で開き、エラー無しで描けたことを確かめた
