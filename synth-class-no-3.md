# Synth Class の実装

### テスト

最終的には次のようなテストとなりました。前項との大きな違いは、一定時間後に停止するという非同期処理の部分を修正したことです。async / await と Promise Object を使って、500ms 後に停止を実行しています。

[https://codesandbox.io/s/wyzx356nv5?module=%2Ftest%2FSynth.spec.js](https://codesandbox.io/s/wyzx356nv5?module=%2Ftest%2FSynth.spec.js)

{% code-tabs %}
{% code-tabs-item title="Synth.spec.js" %}
```javascript
import Synth from '/src/Synth'

import createAudioContext from '/src/WebAudioAPI/createAudioContext'

const audioContext = createAudioContext()
const nextNode = audioContext.destination
const synth = new Synth({ audioContext, nextNode })
const now = audioContext.currentTime

// 再生時に期待する動き
describe('play', () => {
  const options = {
    midiNoteNumber: 70,
    startTime: now,
  }

  it('state', () => {
    synth.play(options)

    const state = synth.oscillatorArray[options.midiNoteNumber].state
    const expectedState = {
      play: true,
      midiNoteNumber: 70,
    }

    expect(state).toEqual(expectedState)
  })

  it('osc', () => {
    const osc = synth.oscillatorArray[options.midiNoteNumber].osc

    expect(!!osc).toBe(true)
  })
})

// 停止時に期待する動き
describe('stop', () => {
  const options = {
    midiNoteNumber: 70,
  }

  it('state', async () => {
    const promise = new Promise((resolve, reject) => {
      setTimeout(() => {
        synth.stop(options)
        resolve()
      }, 500)
    })
    await promise

    const state = synth.oscillatorArray[options.midiNoteNumber].state
    const expectedState = {
      play: false,
      midiNoteNumber: null,
    }

    expect(state).toEqual(expectedState)

    const osc = synth.oscillatorArray[options.midiNoteNumber].osc

    expect(!!osc).toBe(false)
  })

  it('osc', () => {
    const osc = synth.oscillatorArray[options.midiNoteNumber].osc
    expect(!!osc).toBe(false)
  })
})

// すでに停止している音を停止させる時の挙動
describe('double stop', () => {
  const options = {
    midiNoteNumber: 70,
  }

  it('state', () => {
    const state = synth.oscillatorArray[options.midiNoteNumber].state
    const expectedState = {
      play: false,
      midiNoteNumber: null,
    }

    expect(state).toEqual(expectedState)
  })

  it('osc', () => {
    const osc = synth.oscillatorArray[options.midiNoteNumber].osc
    expect(!!osc).toBe(false)
  })
})

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Synth Class 本体

[https://codesandbox.io/s/wyzx356nv5?module=%2Fsrc%2FSynth.js](https://codesandbox.io/s/wyzx356nv5?module=%2Fsrc%2FSynth.js)

{% code-tabs %}
{% code-tabs-item title="Synth.js" %}
```javascript
import createOscillator from '/src/WebAudioAPI/createOscillator'

class Synth {
  constructor({ audioContext, nextNode }) {
    this.audioContext = audioContext
    this.output = nextNode
    this.oscillatorArray = this.createOscillatorArray()
    this.createOscillator = createOscillator({ audioContext })
  }

  createOscillatorArray() {
    const vacantArray = Array.from({ length: 128 })
    return vacantArray.map(o => ({
      state: { midiNoteNumber: null, play: false },
      osc: null,
    }))
  }

  createGain(volume = 0.2) {
    const gainNode = this.audioContext.createGain()
    gainNode.gain.value = volume
    return gainNode
  }

  play({ midiNoteNumber, startTime = 0 }) {
    const osc = this.createOscillator({ midiNoteNumber })
    const gain = this.createGain()
    osc.connect(gain)
    gain.connect(this.output)
    osc.start(startTime)

    const state = {
      play: true,
      midiNoteNumber,
    }

    const item = {
      osc,
      state,
    }

    this.oscillatorArray[midiNoteNumber] = item
    return `play ${midiNoteNumber}`
  }

  stop({ midiNoteNumber, endTime = 0 }) {
    const targetOscillator = this.oscillatorArray[midiNoteNumber].osc
    if (!targetOscillator) {
      return `not ${midiNoteNumber} playing`
    }

    targetOscillator.stop(endTime)

    const state = {
      play: false,
      midiNoteNumber: null,
    }
    this.oscillatorArray[midiNoteNumber].state = state
    this.oscillatorArray[midiNoteNumber].osc = null
    return `stop ${midiNoteNumber}`
  }
}

export default Synth

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 実装のポイント

#### audioContext を外から受け取る

Synth クラスはインスタンスを作成する際に、`audioContext` を受け取ります。これによって、複数のシンセを作成しても、全て同じ `audioContext` から作成することができます。異なる `audioContext` から作成してしまうと時間等が一致しません。

```javascript
const synth = new Synth({ audioContext, nextNode })
```

#### 次に接続する先を外から受け取る

同様に Synth クラスは、次の出力先である `nextNode` も受け取ります。これによって自由なルーティングを実現し、例えば直接 `destination` に出力するのではなく、新たに作ったリバーブモジュールに出力する、といったことが可能になります。

#### oscillatorArray の default 値を設定するメソッド

`createOscillatorArray` メソッドは、`oscillatorArray` の default 値を作成するための関数です。`state` と `osc` というプロパティを item として持つようにします。演奏状態が変更されるたびに、この配列の対応するプロパティの値が変更されます。

```javascript
createOscillatorArray() {
    const vacantArray = Array.from({ length: 128 })
    return vacantArray.map(o => ({
      state: { midiNoteNumber: null, play: false },
      osc: null,
    }))
  }
```

#### createOscillator

{% code-tabs %}
{% code-tabs-item title="Synth.js" %}
```javascript
import createOscillator from '/src/WebAudioAPI/createOscillator'
this.createOscillator = createOscillator({ audioContext })
```
{% endcode-tabs-item %}
{% endcode-tabs %}

`import` している `createOscillator` は、関数を返す関数、つまり高階関数になっています。この関数は、`audioContext` を受け取って、関数を返します。その結果、`this.createOscillator()` と実行することで、高階関数から返ってきて保存されていた関数が実行されます。

次のコードは、`createOscillator` です。先ほど述べたように、これは高階関数なので、実行すると関数を返します。

なぜこのようにする必要があるかというと、`audioContext.createOscillator()` で使用する `audioContext` は引数として一度受ければいいわけですが、`midiNoteNumber` と `wave` は、音程が変更されるたびに引数として受けなくてはいけないからです。こういう場合には高階関数を利用するのがスマートです。

{% code-tabs %}
{% code-tabs-item title="createOscillator.js" %}
```javascript
import convertMidiNoteToFrequency from '/src/Util/convertMidiNoteToFrequency'

const createOscillator = ({ audioContext }) => {
  return ({ midiNoteNumber, wave = 'sine' }) => {
    const osc = audioContext.createOscillator()
    osc.type = wave
    osc.frequency.value = convertMidiNoteToFrequency(midiNoteNumber)
    return osc
  }
}

export default createOscillator

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Synth Class をインスタンス化して使用する

作成した Synth Class をインスタンス化して音を鳴らしてみましょう。きっちりできましたね。

[https://codesandbox.io/s/8n7r4z0yz0](https://codesandbox.io/s/8n7r4z0yz0)

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript

import Synth from '/src/Synth'
import createAudioContext from '/src/WebAudioAPI/createAudioContext'

const audioContext = createAudioContext()
const destination = audioContext.destination

const synth = new Synth({
  audioContext,
  nextNode: destination,
})

const option1 = {
  midiNoteNumber: 75,
  wave: 'sawtooth',
  velocity: 100,
}

synth.play(option1)

const option2 = {
  midiNoteNumber: 75,
}

setTimeout(() => synth.stop(option2), 2000)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

では次に MIDI キーボードを接続して、このブラウザシンセサイザーを演奏できるようにしていきます。そのためには Web MIDI API を使用します。

