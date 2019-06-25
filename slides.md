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

<!-- .slide: data-transition="slide-in none-out" -->

![Audio context](assets/audio-context.png)

---

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 1](assets/audio-context-1.png)

---

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 2](assets/audio-context-2.png)

---

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 3](assets/audio-context-3.png)

---

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 4](assets/audio-context-4.png)

---

## Typical workflow

<!-- .slide: data-transition="none" -->

![Audio context 5](assets/audio-context-5.png)

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
