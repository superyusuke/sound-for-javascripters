# MIDI メッセージを受け取った時の処理を付与する

`MidiInputDevices` は各配列にそれぞれデバイスが入っています。そのデバイスの `onmidimessage` プロパティにハンドラを付与すれば、MIDI メッセージが来た時にそのハンドラを実行してくれます。以下の部分ですね。

```javascript
MidiInputDevices.forEach(o => (o.onmidimessage = getMIDIMessage(synth)))
```

このハンドラには MIDI メッセージが引数として渡されます。ただ少しこの段階では分かりづらいのですが、この getMIDIMessage は単なる関数ではなく高階関数と呼ばれる関数を返す関数ですので、synth という名前で与えているのは MIDI メッセージではありません。これはあくまで関数を返す際に必要な値です。`getMIDIMessage(synth)` を実行して、帰ってきた関数がハンドラとして登録され、そのハンドラに MIDI メッセージが引数として渡されます。

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
import Synth from '/src/Synth'
import createAudioContext from '/src/WebAudioAPI/createAudioContext'
import getMIDIInputDevices from '/src/MIDI/getMIDIInputDevices'
import getMIDIMessage from '/src/MIDI/getMIDIMessage'

const audioContext = createAudioContext()
const destination = audioContext.destination

// synth インスタンスを作成
const synth = new Synth({
  audioContext,
  nextNode: destination,
})

// MIDI device にアクセスして、
// MIDI message を受け取った際に起動させるハンドラを登録する
const start = async () => {
  // MIDI デバイスを取得する
  const MidiInputDevices = await getMIDIInputDevices()
  console.log({ MidiInputDevices })
  if (MidiInputDevices) {
    // 全ての MIDI デバイスへ
    // MIDI メッセージを受け取った際に起動するハンドラを付与する
    // getMIDIMessage がハンドラ
    // ハンドラで使用する synth インスタンスを渡す
    MidiInputDevices.forEach(o => (o.onmidimessage = getMIDIMessage(synth)))
  }
}

start()

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### getMIDIMessage を作る

[https://codesandbox.io/s/km42r386r5?module=%2Fsrc%2FMIDI%2FgetMIDIMessage.js](https://codesandbox.io/s/km42r386r5?module=%2Fsrc%2FMIDI%2FgetMIDIMessage.js)

{% code-tabs %}
{% code-tabs-item title="getMIDIMessage.js" %}
```javascript
const getMIDIMessage = synth => message => {
  const command = message.data[0]
  const midiNoteNumber = message.data[1]
  // command が 128 の時は
  // noteoff 信号が来ているので velocity = 0 をかえす
  // それ以外の時には data[2] に velocity が入っている
  const velocity = command !== 128 ? message.data[2] : 0

  console.log(
    {
      command: message.data[0],
      midiNoteNumber: message.data[1],
      velocity: message.data[2],
    },
    'got midi message!!'
  )

  switch (command) {
    case 144: // noteOn
      if (velocity === 0) {
        const option = {
          midiNoteNumber,
          velocity,
        }
        synth.stop(option)
        return option
      }

      {
        const option = {
          midiNoteNumber,
          wave: 'sawtooth',
          velocity,
        }

        synth.play(option)
        return option
      }

    case 128: // noteOff
      const option = {
        midiNoteNumber,
      }
      synth.stop(option)
      return option

    default:
      return 'default case'
  }
}

export default getMIDIMessage

```
{% endcode-tabs-item %}
{% endcode-tabs %}

まずこれは高階関数になっている点に留意してください。メッセージを受け取った時にならすシンセをまず引数として受け取って、関数を返します。返される関数をは message を受け取って、それを元に条件分岐しシンセを制御する関数です。

構造だけに着目すると以下のようになっています。

```javascript
const getMIDIMessage = synth => message => {
    // message で分岐する処理
}
```

これを `getMIDIMessage(synth)` として実行すると以下のような関数を返します。この関数には引数として渡した synth を使うようになっているわけですね。

```javascript
(message) => {
    // message で分岐する処理
}
```

### message で分岐する処理

さて、受け取った MIDI メッセージの data というプロパティに必要な値が入っています。これは配列で、index 0 にはコマンドが、1 には MIDI Note Number が、それから command 128 以外の時には index 3 にヴェロシティが入っています。

これを元に分岐していくわけです。

{% code-tabs %}
{% code-tabs-item title="MDI/getMIDIMessage.js" %}
```javascript
const command = message.data[0]
const midiNoteNumber = message.data[1]
// command が 128 の時は
// noteoff 信号が来ているので velocity = 0 をかえす
// それ以外の時には data[2] に velocity が入っている
const velocity = command !== 128 ? message.data[2] : 0

```
{% endcode-tabs-item %}
{% endcode-tabs %}

条件分岐はまず command をみて、再生停止を振り分けます

* 144: note on なので再生
* 128: note off なので停止

さらに、144: note on の場合でも velocity が 0 の場合は停止だと判断します。

あとはその判定を元に synth を .play .stop するだけですね。

```javascript
const func = () => {
  switch (command) {
    case 144: // noteOn
      if (velocity === 0) {
        const option = {
          midiNoteNumber,
        }
        synth.stop(option)
        return option
      }

        const option = {
          midiNoteNumber,
          wave: 'sawtooth',
          velocity,
        }

        synth.play(option)
        return option
        
    case 128: // noteOff
      const option = {
        midiNoteNumber,
      }
      synth.stop(option)
      return option

    default:
      return 'default case'
  }
}

```

さてこれで全て実装できました。

