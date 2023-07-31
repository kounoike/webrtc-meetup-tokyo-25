---
theme: seriph
# background: https://source.unsplash.com/collection/94734566/1920x1080
background: /webrtc-image.jpg
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: WebRTC小話あれこれ
fonts:
  # basically the text
#  sans: 'Zen Maru Gothic'
#  serif: 'Zen Maru Gothic'
  # sans: 'Kiwi Maru'
  # serif: 'Kiwi Maru'
  sans: 'Mochiy Pop P One'
  serif: 'Zen Maru Gothic'
# for code blocks, inline code, etc.
  mono: 'Fira Code'
  weights: '400'
---

# WebRTCのこまかい話

WebRTC Meetup Tokyo #25  
こーのいけ

---

# 目次

- Bluetooth ヘッドセットの話
- getUserMedia と enumerateDevices の卵と鶏
- 自分のローカルビデオを一工夫する話
- 背景ぼかし他(media-processors)
- background-blur API

---
layout: cover
background: /bluetooth-image.jpg
---

# Bluetooth ヘッドセットの話

---

# 骨伝導大好き!

- 耳を塞がない
  - インターホンの音とか聞こえる
  - 長時間つけてても耳が痛くならない

TODO: 骨伝導伝道師Slack Emoji
TODO: 骨伝導の写真

#### 原理的には有線でも良いが、ほぼ全ての製品が Bluetooth 接続

---

# Bluetooth ヘッドセットの困ったところ

- マイクを使ったときの音質
- プロファイルの切り替え
- デバイスドライバの不具合が多い（特にmacOS/iOS）

有線骨伝導ヘッドセットが欲しい・・・

---

# Bluetooth ヘッドセットの音質問題

## プロファイルの種類

A2DP, AVRCP, HSP, HFP とか

- A2DP: Advanced Audio Distribution Profile: 音楽再生
- AVRCP: Audio/Video Remote Control Profile: リモコン
- HSP: Headset Profile: 音声通話
- HFP: Hands-Free Profile: 音声通話（HSP の上位互換）

## プロファイルと音質

A2DP はコーデックにもよるがサンプリング周波数が最低でも 48kHz

マイクを使える HSP/HFP はサンプリング周波数が最高でも 16kHz

A2DP と HSP/HFP は同時に使えない→マイクを使うと音質が悪くなる

---

# プロファイルの切り替え

## 開くデバイスによって切り替える（ちょっと前の Windows の話）

- 「ヘッドホン」を開くと A2DP
- 「ヘッドセット」を開くと HSP/HFP

## 自動で切り替わる（今の話）

- 再生デバイスを開いただけなら A2DP で接続
- 録音デバイスを開くと HSP/HFP で接続
- 閉じると A2DP に戻る。

→ WebRTC アプリのミュート実装でうっかりマイクを閉じると大変なことに

（カメラオフは一般的にデバイスを閉じる）

---
layout: cover
background: /egg-chicken.jpg
---

# getUserMedia と enumerateDevices の<br />卵と鶏

---

# デモ

https://gz5yzs.csb.app/

![](/QR.png)

---

# 解説

- getUserMedia でカメラを取得するときにデバイスIDを指定したい
- デバイスIDを取得するために enumerateDevices を使いたい
- 権限が無いと enumerateDevices が中身のないリストを返す
- 権限を取るには getUserMedia の呼び出しが必要　　👈ループ

![](/enumerateDevices.png)


---
layout: cover
background: /localvideo.jpg
---

# 自分のローカルビデオを<br />一工夫する話

デモをもう一度ご覧ください

---

# なぜこうなったか

### 人間は自分の姿を見たとき、鏡の中の自分を見ているのと同じように感じる

→ 左右を反転しておくと良い

```html
<video style="transform: scaleX(-1);">
```

---
layout: cover
background: /blur.jpg
---

# 背景ぼかし他<br /> (media-processors)

<mdi-github /> shiguredo/media-processors
---

# media-processors

## 特徴

- 時雨堂開発のOSSライブラリ
- Soraに依存していない、ベンダーフリー

## 機能

- 仮想背景/背景ぼかし
- 音声ノイズ抑制
- ライト調整

※発表者は時雨堂より委託を請けてmedia-processorsの開発に携わっています。

---

# 仮想背景/背景ぼかし

```mermaid
graph LR;
    MediaStream-->VideoTrack;
    MediaStream-->AudioTrack;
    VideoTrack-->背景ぼかし;
    背景ぼかし-->VideoTrack2;
    VideoTrack2-->MediaStream2;
    AudioTrack-->MediaStream2;
    MediaStream2-->WebRTC;
```

- アプリケーション実装者がライブラリ実装を意識しなくて良い
  - ChromeではMediaStreamTrack Insertable Media Processing using Streams
  - SafariではHTMLVideoElement.requestVideoFrameCallback()
- mediapipe: Googleの謎技術によりtfliteモデルをGPUで推論
  - mediapipeはOSSだがJSにするところは非公開
- ぼかし／背景処理を含めて全てGPUで動かすことで高速動作([shiguredo/media-processors#347](https://github.com/shiguredo/media-processors/pull/347))
  - TODO: Safariのぼかし処理はCPU処理

---

# 音声ノイズ抑制

```mermaid
graph LR;
    MediaStream-->VideoTrack;
    MediaStream-->AudioTrack;
    VideoTrack-->MediaStream2;
    AudioTrack-->ノイズ抑制;
    ノイズ抑制-->AudioTrack2;
    AudioTrack2-->MediaStream2;
    MediaStream2-->WebRTC;
```

- RNNoiseのWasm版
- ノイズ抑制といえば一昔前はRNNoise一強だった
  - RNNoiseの作者はAmazon Chimeへ(VoiceFocus)
- Microsoft主催のDeep Noise Suppression Challenge
  - FRCRN/sudo rm -rf/FullSubNetなど

---

# ライト調整

```mermaid
graph LR;
    MediaStream-->VideoTrack;
    MediaStream-->AudioTrack;
    VideoTrack-->ライト調整;
    ライト調整-->VideoTrack2;
    VideoTrack2-->MediaStream2;
    AudioTrack-->MediaStream2;
    MediaStream2-->WebRTC;
```

- CPU版
  - Efficient Contrast Enhancement Using Adaptive Gamma Correction With Weighting DistributionをZig/Wasmで実装
  - 効き目は弱め
- GPU版(New!)
  - Semantic Guided Low Light Enhancementをtfjs/WebGL backendで推論
  - GPUで推論し描画までGPU（こだわりポイント）
  - 効き目は強め（強弱調整可能）

---
layout: cover
background: /blur-api.jpg
---

# background-blur API

---

# background-blur API

- 最近のWeb会議システムの必須機能: 背景ぼかし
  - Tensorflow.jsやWebAssemblyで実装されている
    - →効率悪いよね
    - →ネイティブ実装してAPI提供したい
  - https://chromestatus.com/feature/5077577782263808
    - Chrome Desktop Dev Trial: 112 / Origin Trial: 114 to 117
    - Consensus & Standardization: Firefox/Safari/Web DevelopersいずれもPositive
---

# OS側の機能

<div class="grid grid-cols-2 gap-4">
  <div>
    <Tweet id="1589891382956609538" scale="0.7" />
  </div>
  <div>
    <Tweet id="1473738646520016911" scale="0.7" />
  </div>
</div>

---

# Windowsの状況

- Windows Studio Effectsの機能の一つ
- NPU搭載が必須
  - ほとんどのPCが対応していない
    - Surface Pro 9 with 5G
    - Windows開発キット 2023
    - ThinkPad Carbo Gen 10
    - AMD Ryzen AI(Ryzen 7040シリーズ)
      - 搭載モデルによりON/OFFされているらしい？
      - Minisforum UM790 Proはダメでした・・・
    - Intel Meteor Lake以降？
    - NVIDIA?

---

# Macの状況

- コントロールセンターで操作
- M1/M2搭載が必須

<div style="height: 50px;">
</div>

# Android/iOS/ChromeOSの状況

### (詳しく調べてない)


---

# ブラウザ側の状況

M2 Macで動作確認したバージョン

|      |状況|Chrome|Safari|Firefox|
|------|----|----|----|----|
|未対応|ぼかしがかからない| |16.5.1| |
|対応|ぼかしがかかる|114| |114.0.2|
|中間地点|ぼかしのON/OFFをJSから取得可能|114(112～flag/114～OT)| | |
|ゴール|ぼかしのON/OFFをJSから設定可能| | | |

---

# 設定・取得の方法


## 設定

```js
stream = await navigator.mediaDevices.getUserMedia({
  video: {
    backgroundBlur: true,
  },
});
// or applyConstraints()
```

## 取得

```js
const track = stream.getVideoTracks()[0];
const settings = track.getSettings();
console.log(settings.backgroundBlur);
```

---
layout: cover
background: /eos.jpg
---

# スライドおわり

---

# おまけ

## セクションタイトルの背景はBing Image Creatorで作りました

・・・なぜ滑り台？
