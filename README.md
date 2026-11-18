# eeg-pipes

eeg-pipes provides TypeScript digital signal processing utilities as pipeable RxJS operators for EEG data in Node.js and the browser. Use it to turn streams of channel readings into epochs, filter signals, and calculate frequency-band power.

## Features

- Group EEG samples into epochs and buffers for downstream processing.
- Apply low-pass, high-pass, band-pass, and notch IIR filters.
- Transform epochs with FFT and calculate power in alpha, beta, delta, gamma, and theta bands.
- Select or remove channels and attach stream metadata.
- Calculate per-channel standard deviation as a signal-quality measure.
- Convert voltage readings to microvolts.

## Installation

Install the library and RxJS in your TypeScript project:

```bash
npm install eeg-pipes rxjs
npm install --save-dev typescript tsx
```

## Usage

Operators consume RxJS Observables. Each input sample contains a `data` array with one reading per channel, a timestamp, and optional `info` metadata such as the sampling rate and channel names.

Save this example as `example.ts`. It generates a single-channel 10 Hz test signal sampled at 250 Hz, groups it into overlapping epochs, and calculates alpha-band power.

```ts
import { from } from "rxjs";
import { epoch, fft, alphaPower } from "eeg-pipes";

const samplingRate = 250;
const startTime = Date.now();

const samples = Array.from({ length: 512 }, (_, index) => ({
  data: [Math.sin((2 * Math.PI * 10 * index) / samplingRate)],
  timestamp: startTime + (index * 1000) / samplingRate,
  info: {
    samplingRate,
    channelNames: ["test"]
  }
}));

from(samples)
  .pipe(
    epoch({ duration: 256, interval: 100, samplingRate }),
    fft({ bins: 256 }),
    alphaPower()
  )
  .subscribe({
    next: (power) => console.log(power),
    error: (error) => console.error(error)
  });
```

Run it with:

```bash
npx tsx example.ts
```

Here, `duration` and `interval` are sample counts: each epoch contains 256 samples, with a new epoch starting every 100 samples. Epoch data is arranged as channels × samples.

For device data, replace `from(samples)` with your EEG Observable. A callback-based device API can be adapted with RxJS `fromEvent`, provided its emitted values match the sample structure above. Set the sampling rate to match your device; the synthetic signal in this example is only a processing demonstration.

## License

MIT. This project is based on [neurosity/eeg-pipes](https://github.com/neurosity/eeg-pipes), originally developed by Neurosity and released under the MIT license. Retain the upstream copyright and permission notices when redistributing its code or substantial portions of it.
