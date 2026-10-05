# ⚡ Tesla Coil Synth Lab

Solid State Musical Coils visualizer — upload any song, it gets transcribed to polyphonic Tesla coil notes and resynthesized live as harsh square-wave + spark-noise, with arc visuals per coil.

Single-file app: just `index.html`. No build, no deps.

## Features

- **Upload MP3 / WAV / OGG / M4A / FLAC** — drag & drop or file picker
- **Built-in demo song** — A-minor coil etude, no upload needed
- **1–4 polyphonic coils (DRSSTC model)** — square + detuned saw + tanh waveshaper + highpassed spark hiss
- **FFT transcription, not sample playback** — original audio is never played, 100% coil synth
- **Canvas visuals** — persistent arc channels per pitch, toroid glow, floor grid, spark particles
- **Controls** — Volume, Sensitivity, Arc length, Coil count
- **Mobile-friendly** — collapsible bottom sheet, big touch transport, responsive coil layout

## Quick start

```bash
# any static server works, e.g:
python3 -m http.server 8000
# open http://localhost:8000
```

Or just double-click `index.html` — but a local server is more reliable for AudioContext in some browsers.

## Usage

1. Click **⬆ Upload MP3 / WAV** (or drop a file on the drop zone) or **▶ Demo song**
2. Wait for decode + transcribe (progress bar shows FFT pass)
3. Press **▶ Play Coils** (Space also toggles play/pause)
4. Tweak live:
   - **Coils 1–4** — synth voices + visuals (file songs re-map existing frames on next Play)
   - **Sensitivity low/med/high** — pitch-pick threshold used at transcribe time
   - **Arc length 20–150%** — visual arc reach
   - **Volume 0–100%**

Best results: melodic / bass-heavy tracks. Dense mixes transcribe to the N loudest notes per ~46 ms frame.

## How it works

1. File decoded via `AudioContext.decodeAudioData` → mono mix
2. `4096-pt FFT / 2048 hop (~46 ms)` with Hann window per frame
3. Harmonic-weighted pitch score (5 harmonics, E1–G6) → top N peaks, neighbour-leakage suppression, voice-continuity assignment, 1-frame blip smoothing
4. Scheduler (`setInterval 40 ms`) drives one voice per coil:
   `square osc + detuned saw → waveshaper (tanh 3.2) → lowpass (freq-tracked) → VCA` + looped noise → highpass → gated hiss, through compressor + slapback delay for room
5. Visuals: each coil holds a strike path for 2–4 pulses then re-strikes same compass heading; heading/length/tint derived deterministically from MIDI note, so same freq = same air channel.

## Tech

- Plain HTML + CSS + JS + Web Audio + Canvas 2D
- Custom radix-2 FFT (no libs)
- Responsive: `pointer:coarse` layout, `visualViewport` resize, iOS audio unlock on touch

## Limits

- Transcription is monophonic-per-coil pitch tracking, not full stem separation — expect artistic interpretation, not perfect covers.
- Large files take a few seconds to transcribe (done on main thread).
- Sensitivity only applies at transcribe time; re-upload to re-transcribe with different value.
