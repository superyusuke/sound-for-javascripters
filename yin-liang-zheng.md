# 音量調整

今までのアプリケーションはあまりに音量が大きすぎました。すみません。そこで、gainNode で音量を調節します。具体的には 1 以下の値を与えることで、音量を下げます。

[https://codesandbox.io/s/18lz7nz9o4](https://codesandbox.io/s/18lz7nz9o4)

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
import convertMidiNoteToFrequency from '/src/convertMidiNoteToFrequency'

const audioContext = new AudioContext()
const destination = audioContext.destination

const MidiNoteArray = [60, 64, 67, 71]

const oscArray = MidiNoteArray.map(frequency => {
  const osc = audioContext.createOscillator()
  osc.frequency.value = convertMidiNoteToFrequency(frequency)

  // 音量調節用の nord
  const gain = audioContext.createGain()

  // 1が default の値なので、それより下げる
  gain.gain.value = 0.2
  // osc -> gain -> destination
  // となるようにつなぐ
  osc.connect(gain)
  gain.connect(destination)

  return osc
})

const now = audioContext.currentTime

oscArray.forEach((osc, index) => {
  osc.start(now + index)
})

setInterval(() => {
  oscArray.forEach(osc => osc.stop())
}, 6000)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### ルーティングのポイント

osc -&gt; gainNode -&gt; destination と順番につなぎます。gainNode を通る際に音量が小さくなるわけですね。

