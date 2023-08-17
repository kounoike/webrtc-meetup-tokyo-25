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

※正確には鏡は奥行きを反転するが、画面の上では左右を反転することに等しい
