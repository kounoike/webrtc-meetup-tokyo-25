---
theme: seriph
# background: https://source.unsplash.com/collection/94734566/1920x1080
background: /MediaStream.jpg
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
title: MediaStreamを一工夫しよう!
fonts:
  # basically the text
#  sans: 'Zen Maru Gothic'
#  serif: 'Zen Maru Gothic'
  # sans: 'Kiwi Maru'
  # serif: 'Kiwi Maru'
#  sans: 'Mochiy Pop P One, Noto Color Emoji'
#  serif: 'Zen Maru Gothic'
# for code blocks, inline code, etc.
  mono: 'Fira Code'
  weights: '400'
---

# MediaStreamを<br />一工夫しよう!

WebRTC Meetup Tokyo #25  
こーのいけ

---

# 自己紹介

## こーのいけ

画像・映像系を中心に色々やってるフリーランスエンジニア

<fa6-brands-x-twitter />ko_noike <fa6-brands-square-github /> kounoike <material-symbols-square-rounded />@kounoike.bsky.social <br />

<div style="height: 30px;"></div>

https://zenn.dev/kounoike/articles/google-meet-bg-features<br />
なぜGoogle Meetの背景ぼかしが最強なのか（一般公開版）

技術評論社 Software Design 2023年5月号 第2特集 「WebAssemblyの可能性を探る」<br />
第3章 「挑戦！ アプリケーションのWebAssembly化」

WebRTC Meetup Online #2<br />
ブラウザがWebカメラであなたの顔を画像として取得するまでの仕組み

---

# ブラウザでカメラ・マイクを使う

### getUserMediaにMediaStreamConstraintsを渡す

```js
const constraint = {
  audio: {autoGainControl: false},  // boolean or MediaTrackConstraints
  video: {width: 640, height: 360}, // boolean or MediaTrackConstraints
};
const stream = await navigator.mediaDevices.getUserMedia(constraint);
```

MediaTrackConstraints:

- AudioTrack関連の制約
- VideoTrack関連の制約
- Image Track（静止画）関連の制約 ⚠️今回は触れない

---

# 目次

- MediaTrackConstraints
  - AudioTrack関係（autoGainControl/echoCancellation/noiseSuppression）
  - VideoTrack関係（background-blur API）
- @shiguredo/media-processorsによる加工

---
layout: cover
background: /MediaTrackConstraints.jpg
---

# MediaTrackConstraints

---

# AudioTrack関連の制約

- deviceId
- フォーマットに関する制約
  - channelCount
  - sampleRate
  - sampleSize
- 特性に関する制約
  - latency
- 音を良くするための制約
  - autoGainControl
  - echoCancellation
  - noiseSuppression

今回は一番下のautoGainControl/echoCancellation/noiseSuppressionについての話をします

---

# VideoTrack関連の制約

- デバイスの選択に関する制約
  - deviceId
  - facingMode
- フォーマットに関する制約
  - width
  - height
  - aspectRatio
  - frameRate
- 画像を良くするための制約
  - backgroundBlur（background-blur API）

今回は一番下のbackgroundBlurについての話をします

---
layout: cover
background: /audio.jpg
---

# 音を良くする話

---

# 実験用音源

## ffmpeg/pythonで生成した音源

![](/test_sounds.png)

---

# 事前調査

## そのまま録音する

<img src="/direct.png" style="height: 300px;" />

倍音成分が混じっている？

---

# オートゲインコントロールの効果

### autoGainControl: true を追加

<img src="/agc.png" style="height: 300px;" />

低音量部が増幅されている

※録音スクリプトの都合上ステレオになっている

---

# echoCancellationの効果

### echoCancellation: true を追加

![](/echocancel.png)

---

# 何が起こったのか

### echoCancellationは本来、音がループしているときのハウリング防止機能

<img src="/sound_loop.png" style="height: 300px;" />

### 綺麗な正弦波だとループと誤判定して消されてしまっていた


---

# noiseSuppressionの効果

### noiseSuppression: true を追加

<img src="/noisesuppress.png" style="height: 300px;">

#### （人工的なノイズで判断すべきではないが）効果は控えめ

---
layout: cover
background: /blur-api.jpg
---

# background-blur API

---

# background-blur API

- 最近のWeb会議システムの必須機能: 背景ぼかし
  - Tensorflow.jsやWebAssemblyなどで実装されている
    - →実行効率悪いよね
    - →ネイティブ実装してAPI提供したい
  - https://chromestatus.com/feature/5077577782263808
    - Chrome Desktop Dev Trial: 112 / Origin Trial: 114 to 117
    - Consensus & Standardization: Firefox/Safari/Web DevelopersいずれもPositive

---

# 設定・取得の方法


## 設定

```js
const stream = await navigator.mediaDevices.getUserMedia({
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

# OS側の機能

<div class="grid grid-cols-2 gap-4">
  <div>
    <Tweet id="1589891382956609538" scale="0.7" />
  </div>
  <div>
    <img src="/ss_applech2.png" style="border: 1px solid black;">
    <a href="https://applech2.com/archives/macos12-monterey-facetime-voice-isolation-and-wide-spectrum-mode.html">
      https://applech2.com/archives/macos12-monterey-facetime-voice-isolation-and-wide-spectrum-mode.html
    </a>
  </div>
</div>

---

# Windowsの状況

- Windows Studio Effectsの機能の一つ
- NPU搭載が必須
  - 現時点では下記のような限られた機種のみ
    - Surface Pro 9 with 5G
    - Windows開発キット 2023
    - ThinkPad Carbon Gen 10（情報全然ない・・・）
    - AMD Ryzen AI(Ryzen 7040シリーズ)
      - 搭載モデルによりON/OFFされているらしい？
      - 私が買ったMinisforum UM790 Proはダメでした・・・
  - Intel Meteor Lake以降？

---

# Macの状況

- FaceTimeの機能として紹介されるが、FaceTime以外でも使える
- Neural Engineを使うため、M1/M2搭載が必須

<div style="height: 50px;">
</div>

# Android/iOS/ChromeOSの状況

### (詳しく調べてない)


---

# ブラウザ側の状況

M2 Macで動作確認したバージョン

|      |状況|Chrome|Firefox|Safari|
|------|----|----|----|----|
|未対応|ぼかしがかからない| ||16.5.1|
|対応|ぼかしがかかる|114|114.0.2| |
|中間地点|ぼかしのON/OFFをJSから取得可能|114†| | |
|ゴール|ぼかしのON/OFFをJSから設定可能| | | |

<div style="text-align: right;">†112～:flag/114～:Origin Trial</div>

---

# 補足

## ノイズキャンセル

Windows Studio Effect、コントロールセンターともに機械学習ベースのノイズキャンセル（Voice Focus/声を分離）をサポート

## アイコンタクト

Windows Studio Effectではさらに視線をカメラに合わせる機能もある


これらの機能は今のところブラウザから制御する話は出ていなさそう？


---
layout: cover
background: /blur.jpg
---

# 背景ぼかし他<br /> (media-processors)

<mdi-github /> shiguredo/media-processors
---

# shiguredo/media-processors

## 特徴

- 株式会社時雨堂開発のOSSライブラリ
- Soraに依存していない、ベンダーフリー

## 機能

- 音声ノイズ抑制
- 仮想背景/背景ぼかし
- ライト調整
  - CPU版
  - GPU版（公開準備中） 
- フェイスフレーミング（開発予定）

※発表者は時雨堂より委託を請けてmedia-processorsの開発に携わっています。

---

# 音声ノイズ抑制

- RNNoiseのWasm版を利用
- MediaTrackConstraintsのnoiseSuppressionより強力
  - その分元音声への影響も強い

<p style="margin-bottom: 0px;">余談</p>

- ノイズ抑制といえば一昔前はRNNoise一強だった
  - RNNoiseの作者はAmazon Chimeへ(VoiceFocus)
- Microsoft主催のDeep Noise Suppression Challenge
  - FRCRN/sudo rm -rf/FullSubNetなど

---

# 仮想背景/背景ぼかし

人物領域の検出→背景のぼかし or 差し替え（上段 元画像）

<div>
<img src="/vbg-original.png" width="320" height="180">
<img src="/vbg-blur.png" width="320" height="180" style="display: inline;">
<img src="/vbg-bgchange.png" width="320" height="180" style="display: inline;">
</div>

画像はぱくたそ(https://www.pakutaso.com/20230429104post-46439.html)より

---

# 仮想背景/背景ぼかし

- アプリケーション実装者がブラウザ差異を意識しなくて良い
  - ChromeではMediaStreamTrack Insertable Media Processing using Streams
  - SafariではHTMLVideoElement.requestVideoFrameCallback()
  - Firefoxは未対応
- mediapipe: Googleの謎技術によりtfliteモデルをGPUで推論
  - mediapipeはOSSだがJSにするところは非公開
- ぼかし／背景処理を含めて全てGPUで動かすことで高速動作([shiguredo/media-processors#347](https://github.com/shiguredo/media-processors/pull/347))
  - TODO: Safariのぼかし処理はCPU処理

---

# ライト調整

暗い画像を明るく補正する（上段: 元画像 下段左: CPU版 右: GPU版）

<img src="/la-original.png">
<img src="/la-cpu.png" style="display: inline;">
<img src="/la-gpu.png" style="display: inline;">

画像はぱくたそ(https://www.pakutaso.com/20211010301post-37460.html)より

---

# ライト調整

- CPU版
  - [Efficient Contrast Enhancement Using Adaptive Gamma Correction With Weighting Distribution](https://ieeexplore.ieee.org/document/6336819)をZig/Wasmで実装
  - 効き目は弱め
- GPU版(Coming Soon!)
  - Semantic Guided Low Light Enhancementをtfjs/WebGL backendで推論
  - GPUで推論し描画までGPU（こだわりポイント）
  - 効き目は強め（強弱調整可能）

---

# まとめ

- MediaTrackConstraintsでOSの機能を利用して「良い」ストリームにする
- 取得したストリームを更に加工して「良い」ストリームにする
- （今回話してないこと）ブラウザに入れる前に加工する
  - 仮想カメラの類
  - 音声フィルタ(Krisp/NVIDIA Broadcast App/VSThost)

→「良い」ストリームになるよう一工夫してからWebRTCの通信に送りましょう！

---
layout: cover
background: /eos.jpg
---

# スライドおわり

---

# おまけ

### 背景・イラスト画像はBing Image Creatorで作りました

・・・なぜ滑り台？
