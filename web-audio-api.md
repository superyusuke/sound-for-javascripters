# Hello, Sound

### Hello, Sound

まずはサイン波を鳴らすだけのシンプルなコードを紹介します。

**Codesandbox** というクラウド上の IDE 兼実行環境に、デモを配置しましたので、以下のリンクから、実行内容を確認できます。 ****

**開くとすぐにかなりの音量がでますので注意してください!!**

[https://codesandbox.io/s/q38kn38rz4](https://codesandbox.io/s/q38kn38rz4)

```javascript
// audio Context の作成
const audioContext = new AudioContext()
// oscillator の作成
const osc = audioContext.createOscillator()
// 波形をサイン派に設定
osc.type = 'sine'
// 周波数を 440 Hz に設定
osc.frequency.value = 440
// 最終出力先を作成
const destination = audioContext.destination
// オシレーターを最終出力先に接続
osc.connect(destination)
// オシレーターを再生する
osc.start()

// 5 秒後に止める
setInterval(() => osc.stop(), 5000)
```

### AudioContext

ここで重要なのは**一行目の `AudioContext`** を作り出す部分です。この `AudioContext` を通して、全ての Web Audio API にアクセスしていきます。例えば 4 行目では `AudioContext` のメソッドである `createOscillator()` を実行し、オシレーターを作成していますね。なお `AudioContext` はブラウザに最初から組み込まれているオブジェクトなので、ウェブブラウザ以外の JavaScript の環境、例えば Node.js の環境では基本的に実行できません。\(npm 等で明示的に用意することはできようです。\)

### Web Audio API

Web Audio API は、ウェブブラウザでサウンドを扱うための API です。本書では主にシンセサイザーを作るために使用しますが、他にも音声の分析、サンプルファイルの再生等も可能です。実際の音楽機器のように単一の機能を持ったモジュール = **Audio Node** をつなげていくことでアプリケーションを作成します。

### audioContext.createOscillator\(\)

`audioContext.createOscillator()` で「オシレーター」を作成します。

オシレーター一つは、一音を鳴らす機能を担当します。そのため、例えばピアノのように複数の音を鳴らす場合には、複数のオシレーターを作成する必要があります。

### オシレーターのメソッド

#### .type

波形のタイプを指定します。文字列で指定できるのは以下の4つです。

* sine
* square
* sawtooth
* triangle

これ以外に Custom 波形も設定できるようですが、本書では扱いません。

#### .frequency.value

周波数を Hz で指定します。音名では指定できませんので、後ほど音名 \(正確には Midi Note Number\) を周波数に変換するメソッドを作成します。なお 440 Hz は、標準位置の A = ラです。

#### .connect\(target\)

オシレーター等、オーディオノードは、他のオーディオノードに `.connect()` で接続できます。

上記サンプルコードでは、`audioContext.destination` という最終出力先に接続しています。ここに最終的につなげることでブラウザから音が再生されます。

#### .start\(\)

オシレーターを再生します。音が出ます。

#### .stop\(\)

オシレーターを停止させます。音が止まります。同時にこのオシレーターは破棄されます。ですので、再度音を出したい場合には、この止まったオシレーターを指定しても存在しませんので、エラーが出ます。正しくは、新たなオシレーターを作成します。

### audioContext.destination

最終出力先です。このノードに接続することで、ブラウザから音を再生することができます。

### コードの全体像

* `new AudioContext()` で audioContect を作成する。ここから全ての Web Audio API の機能にアクセスする。
* `audioContext.createOscillator()` でオシレーターを作成する。その後、`osc` に対して、`.type` で波形の種類を、`.frequency.value` で周波数を設定する。
* 最終出力先を `audioContext.destination` で参照する。
*  `osc.connect(destination)` でオシレーターを最終出力先につなぐ
* `osc.start()` で再生、`osc.stop()` で停止をする。停止した場合、`osc` は即破棄される。再度音を出したい場合には、もう一度 `osc` を作成すること。





