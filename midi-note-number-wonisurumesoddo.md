# MIDI Note Number を周波数に変換するメソッド

オシレーターに対して音程を指定する場合には、周波数で指定する必要がありました。例えば標準の A = ラは 440 Hz なので、以下のように指定しました。

```javascript
osc.frequency.value = 440
```

しかし、周波数での指定は、直感的ではありません。これから作ろうとするシンセサイザーは、平均律、つまりドレミファソラシドに則ったものなので、音名で指定した方がわかりやすいでしょう。そのため、音名を引数として与えると、それに対応する周波数を返すメソッドを作成してきます。

### MIDI Note Number とは

MIDI Note Number とは、後述する MIDI という音楽機器の通信プロトコルの中で、音名に対してふられた番号です。MIDI Note Number は 0 から 128 まであり、0 が一番低く、128 が一番高くなっています。

{% embed url="https://newt.phys.unsw.edu.au/jw/notes.html" %}

例えば 440Hz = ラは、上記表で確認していただきたいのですが、MIDI Note Number でいうと 69 番だということがわかります。一オクターブ上のラは 81 番で 880 Hz ですね。一オクターブ上がると、周波数は 2 倍に、MIDI Note Number は 12 個増えます。

では、この MIDI Note Number を周波数に変換するにはどうしたらいいでしょうか。

### MIDI Note Number を周波数に変換する式

`m` を MIDI Note Number とすると、それに対応する周波数 `fm` は、以下のような式で表すことができるそうです。これに関しては物理と数学の問題ですから、詳しく説明することなく援用します。

`fm = 2 ^ ((m − 69) / 12) * 440`

#### メソッド

以下のメソッドを使って、MIDI Note Number を与えると、きっちり表どおりの周波数が帰ってきますね。\(若干違うのは、表の方が値をある程度切りが良いものに変更しているのでしょうか。とにかく実用上は問題ない範囲で正確です。\)

[https://codesandbox.io/s/7jmpkqnjrx](https://codesandbox.io/s/7jmpkqnjrx)

```javascript
const convertMidiNoteToFrequency = midiNoteNumber =>
  2 ** ((midiNoteNumber - 69) / 12) * 440

console.log(convertMidiNoteToFrequency(69), 69)
console.log(convertMidiNoteToFrequency(60), 60)
console.log(convertMidiNoteToFrequency(67), 67)

```

### 和音を鳴らしてみる

[https://codesandbox.io/s/7jmpkqnjrx](https://codesandbox.io/s/7jmpkqnjrx)

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
import convertMidiNoteToFrequency from '/src/convertMidiNoteToFrequency'

const audioContext = new AudioContext()
const destination = audioContext.destination

// オシレーターを 4 つ作る
const osc1 = audioContext.createOscillator()
const osc2 = audioContext.createOscillator()
const osc3 = audioContext.createOscillator()
const osc4 = audioContext.createOscillator()

// CM7 になるように、Midi Note Number を与える
osc1.frequency.value = convertMidiNoteToFrequency(60)
osc2.frequency.value = convertMidiNoteToFrequency(64)
osc3.frequency.value = convertMidiNoteToFrequency(67)
osc4.frequency.value = convertMidiNoteToFrequency(71)

osc1.connect(destination)
osc2.connect(destination)
osc3.connect(destination)
osc4.connect(destination)

// 現時点の時間を取得
const now = audioContext.currentTime

// 一秒ずつずらして再生
osc1.start(now)
osc2.start(now + 1)
osc3.start(now + 2)
osc4.start(now + 3)

// 止める
setInterval(() => {
  osc1.stop()
  osc2.stop()
  osc3.stop()
  osc4.stop()
}, 5000)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="convertMidiNoteToFrequency.js" %}
```javascript
const 
 = midiNoteNumber =>
  2 ** ((midiNoteNumber - 69) / 12) * 440

export default convertMidiNoteToFrequency
```
{% endcode-tabs-item %}
{% endcode-tabs %}

先ほど作った MIDI Note Number を周波数に変換する関数を使って、一つ一つのオシレーターに異なる周波数を与えることで和音を出しています。

新しい要素として、`audioContext.currentTime` があります。これは現在の時間を数値で返してくれるものです。これを利用して、各オシレーターを 1 秒ごとずらして再生しています。

ただ、繰り返しが多く、かなり素人っぽいコードなので、もう少しスマートにしてみます。具体的には、和音として再生したい MIDI Note Number を配列に入れて、その配列からオシレーターを作成します。

### リファクタリングされたコード

配列を活用することで、繰り返しの記述をなくしました。

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
import convertMidiNoteToFrequency from '/src/convertMidiNoteToFrequency'

const audioContext = new AudioContext()
const destination = audioContext.destination

// 再生したい MIDI Note Number が含まれた配列を作る
const MidiNoteArray = [60, 64, 67, 71]

// MIDI Note Number の配列をもとに、
// 希望する周波数を再生するオシレーターが含まれる配列を作成
// 最終アウトプットにも接続すること
const oscArray = MidiNoteArray.map(frequency => {
  const osc = audioContext.createOscillator()
  osc.frequency.value = convertMidiNoteToFrequency(frequency)
  osc.connect(destination)
  return osc
})

// 現時点の時間を取得
const now = audioContext.currentTime

// 一秒ずつずらして再生
oscArray.forEach((osc, index) => {
  osc.start(now + index)
})

// 止める
setInterval(() => {
  oscArray.forEach(osc => osc.stop())
}, 6000)

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 音量が大きすぎて割れている問題がある

うすうすみなさんお気づきだと思いますが、実は各音量が多すぎて、サウンドに歪みが生じています。これを修正するために、次の項では音量の調整を行います。

