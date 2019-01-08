# 完成形と改良の余地

[https://codesandbox.io/s/v3y2xvo6wl](https://codesandbox.io/s/v3y2xvo6wl)

ついに完成しましたね。MIDI キーボードを繋いで演奏すればあなたが作ったシンセがなります！

### 改良の余地

まだまだ改良の余地があります。それを列挙し、さらにそのための方向性と手法も紹介します。

### MIDI CC を受け取る

ピッチベンドやモジュレーションホイールの値を受け取って、シンセに変化を与えたいはずです。その場合には command で分岐します。具体的な値はまだ自分もよくわかっていないのですが、console に command を出力して、自分のコントローラーを動かした時のコマンドを確認していけばいいでしょう。

### ADSR の実装

音量やピッチやフィルターといった値は、.value で直接指定するだけではなく `setValueAtTime` や `linearRampToValueAtTime` を使って時間変化させることができます。これに関しては以下の記事が詳しいので参考にしてください。

{% embed url="https://www.keithmcmillen.com/blog/making-music-in-the-browser-web-audio-midi-envelope-generator/" %}

### モジュラーシステム化

今は一台のシンセしか使わない前提の構成になっていますが、できれば複数のシンセやフィルター、シーケンサーといったものをモジュールとして作り、つなげていきたいものです。

現時点でも、次につなぐ先を指定できるので、同じインターフェイスのエフェクト等を作っていけば、つなげていけます。

```javascript
const synth = new Synth({
  audioContext,
  nextNode: destination,
})
```

最終的にはインターフェイスを統一して、いろんな人がモジュールを npm パッケージとして公開する、なんてことになっていけばいいなと思っています。

ではお疲れ様でした！

