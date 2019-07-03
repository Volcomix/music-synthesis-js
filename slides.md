<!-- .slide: class="title" data-background-color="#2a2e33" data-background-image="assets/mixer-background.jpg" data-background-opacity="0.3" -->

# Music Synthesis

### in JavaScript

<small>  
Sébastien Jalliffier Verne  
https://github.com/volcomix/music-synthesis-js
</small>

Notes:
Today I would like to give you an overview of sound synthesis, maybe music creation, and what kind of powerful things
you can do right inside your browser.

---

## What is the Web Audio API?

![](assets/high-level-api.jpg) <!-- .element: width="40%" -->

A high-level JavaScript API for processing and synthesizing audio

Notes:
The Web Audio API is a high-level JavaScript API for processing and synthesizing audio in web applications.
Or if you are not familiar with sound synthesis, you can see it as a low-level API to create sounds!

---

## What can I use it for?

![](assets/game.jpg) <!-- .element: width="40%" style="margin: 0" -->
![](assets/daw.png) <!-- .element: class="plain" width="40%" style="margin: 0" -->
![](assets/art.jpg) <!-- .element: class="plain" width="40%" style="margin: 0" -->

Notes:

- The goal of this API is to include capabilities found in modern game audio engines
- And some of the mixing, processing, and filtering tasks that are found in modern desktop audio production applications.
- You can also make audiovisual art by combining music or sounds produced with Web Audio with visuals made with 2D canvas graphics, 3D WebGL, or SVG.

---

## Where can I use it?

![](assets/caniuse.png)

---

## How does it work?

![](assets/web-audio-vs-guitar-effects.jpg) <!-- .element: class="plain" -->

Notes:

At the heart of the Web Audio API is a number of different audio inputs, processors, and outputs,
which you can combine into an audio routing graph that creates the sound you need.

---

<!-- .slide: data-background-color="#2a2e33" -->

## Workshop

https://codesandbox.io/s/github/volcomix/coder-synth

---

Your code goes here

![](assets/directories.png)

---

You need an AudioContext and a destination

```js
const audioContext = new AudioContext()
const destination = audioContext.destination
```

![](assets/provided.jpg) <!-- .element: class="plain fragment" data-fragment-index="1" width="60%" -->

---

Adding a track for your instrument

```js
// src/music/songs/Workshop1907.js

import Song from '../common/Song'
import Oscillator from '../instruments/demo/Oscillator'

export default class Workshop1907 extends Song {
  tempo = 140
  notesPerBeat = 2
  tracks = [
    {
      instrument: new Oscillator(this.audioContext, this.destination),
    },
  ]
}
```

---

## Let's make some noise!

![](assets/create-sound.jpg) <!-- .element: class="plain" width="90%" -->

---

## Oscillator

![](assets/oscillator.png) <!-- .element: width="90%" style="margin: -18% 0" class="plain" -->

```js
// src/music/instruments/Oscillator.js

import Instrument from '../../common/Instrument'

export default class Oscillator extends Instrument {
  start() {
    this.oscillator = this.audioContext.createOscillator()
    this.oscillator.type = 'sine' // Or square, triangle, sawtooth, custom
    this.oscillator.frequency.value = 440
    this.oscillator.connect(this.destination)
    this.oscillator.start()
  }

  stop() {
    this.oscillator.stop()
  }
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/1" data-background-interactive -->

---

Playing some notes

```js
// src/music/songs/Workshop1907.js

{
  instrument: new Oscillator(this.audioContext, this.destination),
  notes: `
    C-4 --- E-4 --- F#4 --- G-4 ---
    --- --- --- --- --- --- --- OFF
  `,
}

// src/music/instruments/Oscillator.js

export default class Oscillator extends Instrument {
  ...

  noteOn(noteFrequency, time) {
    this.oscillator.frequency.setValueAtTime(noteFrequency, time)
  }

  noteOff(time) {
    this.oscillator.frequency.setValueAtTime(0, time)
  }
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/2" data-background-interactive -->

---

## Amplitude

![](assets/amplitude.png) <!-- .element: style="margin: -15% 0" class="plain" -->

```js
start() {
  this.oscillator = this.audioContext.createOscillator()

  this.gain = this.audioContext.createGain()
  this.gain.gain.value = 0.5

  this.oscillator.connect(this.gain)
  this.gain.connect(this.destination)

  this.oscillator.start()
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/3" data-background-interactive -->

---

Scheduling some effects

<pre><code class="js" data-line-numbers="1,9-14,17-21">// src/music/songs/Workshop1907.js

{
  instrument: new Amplitude(this.audioContext, this.destination),
  notes: `
    C-4 --- E-4 --- F#4 --- G-4 ---
    --- --- --- --- --- --- --- OFF
  `,
  effects: {
    gain: `
    056 --- --- --- --- --- 255 ---
    --- --- --- --- --- --- --- 000
    `,
  },
}

// src/music/instruments/Amplitude.js

fxGain(gain, time) {
  this.gain.gain.linearRampToValueAtTime(gain / 255, time)
}
</code></pre>

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/4" data-background-interactive -->

---

## Envelope

![](assets/envelope.png) <!-- .element: class="plain" -->

---

```js
noteOn(noteFrequency, time) {
  this.oscillator.frequency.setValueAtTime(noteFrequency, time)

  // Attack
  this.gain.gain.setValueAtTime(0, time)
  this.gain.gain.linearRampToValueAtTime(1, time + 0.2)

  // Decay
  this.gain.gain.linearRampToValueAtTime(0.5, time + 0.4)
}

// Sustain

noteOff(time) {
  // Release
  this.gain.gain.setValueAtTime(0.5, time)
  this.gain.gain.linearRampToValueAtTime(0, time + 0.5)
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/5" data-background-interactive -->

---

## Low-frequency oscillator (LFO)

![](assets/lfo.png) <!-- .element: class="plain" width="50%" -->

---

```js
start() {
  this.lfo = this.audioContext.createOscillator()
  this.lfo.frequency.value = 1

  this.oscillator = this.audioContext.createOscillator()
  this.oscillator.frequency.value = 440

  this.modulationGain = this.audioContext.createGain()
  this.modulationGain.gain.value = 50

  this.lfo.connect(this.modulationGain)
  this.modulationGain.connect(this.oscillator.detune)
  this.oscillator.connect(this.destination)

  this.lfo.start()
  this.oscillator.start()
}

stop() {
  this.lfo.stop()
  this.oscillator.stop()
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/6" data-background-interactive -->

---

## Frequency Modulation (FM)

![](assets/fm.png) <!-- .element: class="plain" width="50%" -->

---

```js
start() {
  this.oscillator1 = this.audioContext.createOscillator()
  this.oscillator1.frequency.value = 220

  this.oscillator2 = this.audioContext.createOscillator()
  this.oscillator2.frequency.value = 440

  this.modulationGain = this.audioContext.createGain()
  this.modulationGain.gain.value = 50

  this.oscillator1.connect(this.modulationGain)
  this.modulationGain.connect(this.oscillator2.frequency)
  this.oscillator2.connect(this.destination)

  this.oscillator1.start()
  this.oscillator2.start()
}

stop() {
  this.oscillator1.stop()
  this.oscillator2.stop()
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/7" data-background-interactive -->

---

## Additive synthesis

![](assets/additive.png) <!-- .element: style="margin-top: -5%" class="plain" -->

---

```js
start() {
  this.oscillator1 = this.audioContext.createOscillator()
  this.oscillator1.frequency.value = 220

  this.oscillator2 = this.audioContext.createOscillator()
  this.oscillator2.frequency.value = 440

  this.gain1 = this.audioContext.createGain()
  this.gain1.gain.value = 0.5

  this.gain2 = this.audioContext.createGain()
  this.gain2.gain.value = 0.5

  this.oscillator1.connect(this.gain1)
  this.oscillator2.connect(this.gain2)
  this.gain1.connect(this.destination)
  this.gain2.connect(this.destination)

  this.oscillator1.start()
  this.oscillator2.start()
}

stop() {
  this.oscillator1.stop()
  this.oscillator2.stop()
}
```

---

<!-- .slide: data-background-iframe="https://volcomix.github.io/coder-synth/Demo/7" data-background-interactive -->

---

## Resources

<small>https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API</small>  
<small>https://www.soundonsound.com/techniques/synth-secrets-all-63-parts-sound-on-sound</small>  
<small>https://learningsynths.ableton.com</small>  
<small>https://audionodes.com/online</small>

---

## Web Audio API

<small>See [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)</small>

Audio routing graph

Audio nodes
<small>See [MDN](https://developer.mozilla.org/en-US/docs/Web/API/AudioNode#Description)</small>

Audio params

Scheduling

---

## Concepts

- Audio context
  <small class="fragment" data-fragment-index="1">➔ is the place where audio is operated</small>
- Audio nodes
  <small class="fragment" data-fragment-index="2">➔ are basic elements of audio</small>
- Modular routing
  <small class="fragment" data-fragment-index="3">➔ connects nodes with each other</small>
- Audio routing graph
  <small class="fragment" data-fragment-index="4">➔ the network of audio nodes</small>

---

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context](assets/audio-context.png)

--

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 1](assets/audio-context-1.png)

--

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 2](assets/audio-context-2.png)

--

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 3](assets/audio-context-3.png)

--

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 4](assets/audio-context-4.png)

--

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 5](assets/audio-context-5.png)

---

## General audio graph definition

- AudioContext
- AudioNode
- AudioParam

---

## Defining audio sources

- OscillatorNode
- AudioBuffer
- AudioBufferSourceNode
- MediaElementAudioSourceNode
- MediaStreamAudioSourceNode

---

## Defining audio effects filters

- BiquadFilterNode
- ConvolverNode
- DelayNode
- DynamicsCompressorNode
- GainNode
- WaveShaperNode
- PeriodicWave
- IIRFilterNode

---

## Defining audio destinations

- AudioDestinationNode
- MediaStreamAudioDestinationNode

---

## Data analysis and visualization

- AnalyserNode

---

## Splitting and merging audio channels

- ChannelSplitterNode
- ChannelMergerNode

---

## Audio spatialization

- PannerNode
- StereoPannerNode

---

## Audio processing in JavaScript

TBD

---

Simple example of modular routing

![Simple Modular routing](assets/modular-routing1.png)

```js
const context = new AudioContext()

function playSound() {
  const source = context.createBufferSource()
  source.buffer = dogBarkingBuffer
  source.connect(context.destination)
  source.start(0)
}
```

---

More complex example of modular routing

![More complex modular routing](assets/modular-routing2.png)

---

Modular routing with one oscillator modulating the frequency of another

![Modular routing with FM](assets/modular-routing3.png)

---

```js
function setupRoutingGraph() {
  const context = new AudioContext()
  // Create the low frequency oscillator that supplies the modulation signal
  const lfo = context.createOscillator()
  lfo.frequency.value = 1.0
  // Create the high frequency oscillator to be modulated
  const hfo = context.createOscillator()
  hfo.frequency.value = 440.0
  // Create a gain node whose gain determines the amplitude of the modulation signal
  const modulationGain = context.createGain()
  modulationGain.gain.value = 50
  // Configure the graph and start the oscillators
  lfo.connect(modulationGain)
  modulationGain.connect(hfo.detune)
  hfo.connect(context.destination)
  hfo.start(0)
  lfo.start(0)
}
```

---

## SYNTHESIS BASICS

Oscillator
<small>Frequency</small>

Gain
<small>Amplitude</small>

Biquad filter
<small>Lowpass, highpass, bandpass, ...</small>

Envelope

Frequency Modulation

LFO
