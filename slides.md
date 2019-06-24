# Music Synthesis

### in JavaScript

<small>  
Sébastien Jalliffier Verne  
[github.com/volcomix](https://github.com/volcomix)
</small>

---

## Web Audio API

<small>See [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)</small>

Audio routing graph

Audio nodes  
<small>See [MDN](https://developer.mozilla.org/en-US/docs/Web/API/AudioNode#Description)</small>

Audio params

Scheduling

---

## Web Audio API - Concepts

- Audio context
- Audio nodes
- Modular routing
- Audio routing graph

---

## Web Audio API - Usage

1. Create audio context
2. Inside the context, create sources
3. Create effect nodes (reverb, biquad filter, panner, compressor, ...)
4. Connect the sources up to the effects, and the effects to the destination

![Audio context](assets/audio-context.png)

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
