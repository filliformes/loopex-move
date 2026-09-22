# Loopex

**A 16-track live tape looper for the Ableton Move, built as a Schwung *Overtake* module.**

Loopex turns the Move into an asynchronous, tape-flavoured looping instrument: sixteen
stereo loops on the left pads — each with **up to four independent playheads** — a bank of
momentary/latchable glitch FX on the right pads, two send buses drawing on a 24-effect
Palette engine plus four reverbs, a full record-path tape machine, session save/load,
and a MIDI-keyboard polyphony layer.

> Inspired by the 1010music BlackBox looper workflow, and by the sound of Kinotone Ribbons,
> Chase Bliss Blooper / Mood MK2, and Microcosm-style glitch. Interaction ideas borrowed
> from the norns looper world (*wrms*, *cranes*, *oooooo*, *samsara*, *nydl*, *otis*).

- **Module:** `loopex` · **Name:** Loopex · **Type:** Overtake (Schwung) · **API v2**
- **Format:** 44100 Hz, 128-frame blocks, stereo · **Version:** 0.9.2 · **Manual:** [online](https://filliformes.github.io/loopex-move/) · [markdown](docs/MANUAL.md) · **License:** GPL-3.0

---

## What it does

### 16 stereo tape loops
Each left pad is an independent, free-running stereo loop (up to 45 s). Record, play, pause
and overdub live; every loop has its own speed, tone, FX sends, amp envelope and playheads.

- **Overdub modes:** Replace · **Multiply** (default; endless layering with decay) · Disintegration
  (the loop's FX are re-applied each pass, so it slowly falls apart).
- Playback speed is tinted onto the pad LED (blue ½× · green 1× · yellow 2×).

### Up to 4 playheads per loop
Every loop can be read by **four independent heads at once**, each with its own mode and
speed — one recording becomes a canon, a drone, or a ping-ponging texture.

- Modes: **Off · Fwd · Bwd · Ping** (ping-pong) **· Jump** (leaps to a random spot at random intervals, crossfaded).
- Per-head speed (0.25×–4×); heads sum with `1/sqrt(n)` normalisation so stacking stays sane.
- Turning a head on **always restarts it at the loop start**.
- Head 1 is the main head — Scatter, Seed, scrub and Jump all drive it.
- In the Playheads page, **touch a head's knob and the jog wheel moves that head** along the waveform.

### The five loop pages
Navigate with **Down** (next) and **Up** (previous).

| Page | Knobs |
|------|-------|
| **P1 · Loop** | **Speed** · Filter · Pan · Volume · Start · End · Reverse · **Send A** |
| **P2 · Texture** | **Pitch** · **Reso** · Sat · Comp · Wow/Flutter · Scatter · **Seed** · Send B |
| **P3 · Tone** | Studer **Bass · MidF · MidGain · Treble** · Tilt · Attack · Decay · **Heads ▸** |
| **P4 · Playheads** | H1 mode/speed · H2 · H3 · H4 |
| **P5 · HeadMix** | per-head **Vol/Pan** — H1 (mirrors P1 Vol/Pan) · H2 · H3 · H4 |

- **Speed** — playback rate, ±2 octaves in 0.1-semitone steps (pitch and tempo together, like tape).
  Reads are 4-point Hermite with a rate-aware anti-imaging filter below 0.75×; the sampler
  Characters replace that with linear or drop-sample reads, which is where their grit comes from.
- **Pitch** — an independent shift, −24 to +24 semitones, tempo untouched: a Signalsmith Stretch
  phase-vocoder shifter. It runs on its own worker thread (`lpx-ps`), off the audio callback, and
  its latency (61 ms including the hand-off queue) is cancelled by nudging the playheads, so the
  loop stays in time.
- **Seed** — a Smack-style *seeded slice re-order* (2/4/8/16 slices, some reversed). The knob
  *is* the seed: every position is a different reproducible mangle, click-free.
- **Scatter** — stochastic slice jumps, crossfaded.
- **DJ Filter + Reso** — continuous LP/HP sweep with resonance, smoothed over ~10 ms.
- **Attack / Decay** — per-loop amplitude envelope (3 ms → 3 s / 5 s), used on trigger,
  mute, pause and stop.

### Two send buses — the Palette engine
Send A and B each select from **29 effects** (Off + 24 Palette effects + four reverbs):
Drive, Sweeten, Fuzz, Howl, Fold, Swell, Doubler, Vibrato, Phaser, Tremolo, Pitch, Shift,
Cascade, Reels, Collage, Reverse, Space, Bloom, Filter, Squash, Cassette, Broken,
Interference, Halo, **Plate**, **Quartz**, **Prism**, **Veil** — each with Amount / Macro / Drift.

The last four are proper reverbs, and the last three come from the standalone instruments:

| | Tank | Amount | Macro | Drift |
|---|---|---|---|---|
| **Plate** | Dattorro plate | decay | pre-delay | damping |
| **Quartz** | 8-line Hadamard FDN, dual-band damping | room → hall | dark → bright | still → swimming |
| **Prism** | same tank, frequency-dependent decay | decay | lows ring ⇄ highs shimmer | crossover + movement |
| **Veil** | Householder FDN, modulated diffusers | size + tail | dark → bright | movement + colour |

Every effect is loudness-matched to the dry signal it replaces (the Character family used to sit
many dB apart), and **switching effects morphs** — the outgoing one fades as the incoming one fades
in — so sweeping through the list never clicks. The PalFX punch pad behaves the same way.

### Punch-in FX (right 16 pads)
Sixteen momentary effects over a 2-second capture ring, in four families —
**Loops** (1/12 · 1/16 · short · **Chop**, with 32 rhythmic patterns drawn from Signal),
**Grains** (Haze · Mosaic · Smear · Strum), **Pitch** (Oct− · Oct+ · Glide · Shimmer) and
**Time** (Stretch · Freeze · Reverse · **PalFX**, one Palette effect as a punch: FX / Amount /
Macro / Drift, default Space) — **up to 5 stacked in series**. Slice effects auto-pan in
sync with their rate, and every slot loudness-matches its wet to the dry it replaces. The grain
effects draw on a 16-grain pool with voice stealing and window-power normalisation.
**Oct+, Oct−, Shimmer and Chop are spread to stereo by a microshifter**, the same widener
Stretch and Freeze use.
Hold a pad to apply; **knobs 5–8 edit the held effect's four parameters** live, and
**pad pressure** drives a per-effect expression (subdivide, density, glide, freeze, rate…)
shown in the footer. **Shift + pad latches** it on hands-free; **Shift while holding** latches
it exactly as it is, pressure included. **Undo + pad** resets a pad to its defaults.

### FX sequencer (✕ button)
One shared 16-step pattern of punch pads, modelled on the Polyend MESS. **Tap ✕** to run or
stop; **hold ✕** to see the pattern on the step buttons (orange = step, dim = extension,
white = playhead) and its page: Run · Speed (1/32 … 1 beat) · Length · Chance · Gate ·
Swing · Direction · Clear. **✕ + pad(s) + step** writes up to five pads into a step with
their knobs and pressure locked; **✕ + step** clears it; **✕ + step + later step** extends
it; **✕ + held step + knob 4** sets that step's play chance (Always, 10–90 %, Like Last,
Play X Skip Y) and **knobs 5–8** edit its locks. Play restarts the pattern.

### Tape transport (◀ ▶)
Hold **Left** and the whole master brakes to a stop in about three seconds; hold **Right**
and it winds up to a tone. Release and it eases back to 1×. The glide is linear in
semitones and the last octaves of a stop fade to silence.

### The Input (track button 1, two pages)
Everything you record passes through here first, so set it before you play — it is baked into the
take. **Input Tape** is the machine: Mon (monitor level) · Tape · Trim · Drive · Wow · Flut · Hiss ·
Gen. The 13 models are `Tapeless · Clean · Cass1 · Cass2 · VHS1 · VHS2 · Reel15 · Reel7 · Reel3 · 4trk ·
Porta · Dub · Warp`, default `Clean`. Each model's high-frequency loss is derived from its head gap
and tape speed, with a speed-scaled head bump, so the three reel speeds genuinely differ; **VHS1** is
the Hi-Fi track (an FM carrier with its companding noise reduction, bright but pumping) and **VHS2**
the linear edge track, slow and dark; **Gen** re-applies the machine's own loss filter once per pass.
**Tapeless** bypasses the machine — no hiss floor, no head loss or bump — but every knob still works,
and Drive pushes up to +24 dB into the curve on every model.

**Input EQ** (page 2): LoCut · HF · LowF · Low · Mid · MidF · High · HiF. A blend of the Studer 962
presence band and the Tascam 424 MkII/MkIII channel EQ (owner's manual p.11/36/45): ±15 dB 2nd-order
shelves with a sweepable low corner (**LowF** 40–400 Hz, default 100 Hz — the 424's point) and a high
corner 3–15 kHz (default 10 kHz), a ±12 dB peak sweepable 150 Hz–7 kHz, a 20–800 Hz low cut and a
2–20 kHz tape rolloff (HF: turn *down* to roll off). Every sweep is log, so each octave gets the same
knob travel.

### Dynamic sampling (Capture button)
The input plays the sampler. Turn **Dyn** (K1) to a mode — or **long-press Capture** to toggle it;
the button blinks while it listens — and every phrase you play is captured into a pad by itself: an
ordinary recording, so it gets the tape stage, the loop pages, the sends, and its LED goes red then
green like any take. What was on the pad is *replaced*, never stacked (Chase Bliss Onward);
**Spread** is how many pads the sampler rotates through from the selected one — 1 replaces the same
pad every time, 16 is a rolling memory of the last sixteen phrases (AC Noises Continua's
*dimension*, with sixteen layers). Muted pads are skipped, and a pad that already holds a loop is
**never overwritten until you say so**: when the range is full the menu opens and asks
`ALL PADS FULL: OVERWRITE?` — K8 lets it carry on, K5 or Back turns Dynamic off.

| Knob | |
|---|---|
| **Dyn** | Off · **Level** (a sound above the threshold) · **Onset** (a transient, however loud the sustain under it) · **Phrase** (starts on sound, stops at the next gap) · **Clock** (every Size, on Move's tempo) |
| **Sense** | the mode's threshold, −60 → −6 dBFS |
| **Size** | 1/16 … 8 bars at Move's tempo, or **Free** — record until 200 ms of quiet (default) |
| **Spread** | 1–16 pads from the selected one (default 16) |
| **Error** | how much of each new capture is randomised — 0 faithful, 1 a new creature every time |
| **Sustn** | how long a capture plays before it pauses itself (its own Decay fades it): 1 → 60 s, top = forever |
| **RndPad · RndAll** | randomise every loop-page parameter of the selected pad / of all sixteen, no confirmation |

A 100 ms pre-roll means a capture keeps the transient that triggered it. **Randomise** is musical by rule: Speed and Pitch move only as *complementary musical intervals* (the heard
transposition and the tempo ratio are both octaves, fifths or major/minor thirds — never more than two
octaves — and a third of the time nothing moves),
playhead speeds are never touched, heads 2–4 are switched on or off and only an active one gets a
random pan, the loop's Volume is left alone, and sends, Comp and Sat never go past 60 %.

### Drift — a global evolving memory (Sample button)
A drifting-delay memory station inspired by **Soma COSMOS**, on the **Sample** button. Four
coprime-length delay lines, each read at a slowly drifting tap, feed back through a matrix
that morphs from self-feedback to a normalised **Hadamard** cross-mix. The loop mix feeds it,
the memory recirculates, and the coprime lengths plus per-line async LFOs keep it from ever
landing on an exact repeat. It runs in the master chain just before the pump, tanh-limited so
high feedback sustains without runaway.

- **Drift** feed level · **Rate** tap-drift speed · **Size** tap length (shimmer → long hall)
  · **FBk** memory sustain (below unity fades, near unity holds) · **Supr** loud input erases
  old memory (play over to replace) · **Blur** self-feedback → full cross-mix · **Damp** tail
  high-frequency damping · **Mix** wet into the master.
- **FBk** is capped below unity, so the tail always fades: up to a few minutes at maximum. Leave everything
  paused and a **silence bleed** clears an abandoned tail after about eight seconds with no input.
- Drift and Mix start at zero (silent until dialled in); ~1.7 MB of memory; saves with the session.

### Sessions
**64 numbered slots**, saved and loaded from the **≡ (Menu)** button, each named
`slot_YYYYMMDD_HHMM`; saving over a used slot asks for confirmation. Settings *and*
recorded audio are stored — the audio as plain **16-bit stereo WAV** files (`sessionNN/sNN_loopNN.wav`, one per
loop) that you can copy straight off the Move — the folder shows up in the Schwung Manager's file
browser (`move.local:7700` → Files → `data/UserData/UserLibrary/Loopex`); sessions from before 0.9.2
are moved there and still load.
**Clear** (K5 on the Sessions page) wipes all sixteen loops *and* resets every loop's settings after
a confirmation (the global pages — Output, Input, Drift… — are untouched); **Reset** (K6) returns every *setting* of the selected loop to factory defaults — the audio
and its playback are untouched — also after a confirmation; all disk work runs on a `SCHED_OTHER` worker thread (`lpx-sio`) pinned
to cores 0–2, never on the audio callback. A second worker, `lpx-ps`, runs the per-loop pitch
shifters the same way. Sessions live in
`/data/UserData/UserLibrary/Loopex/` so reinstalls keep them.

### Perform, MIDI and I/O
- **Perform menu (two pages):** page 1 — Stumble (probabilistic step glitch), Jump and **Scan**
  (doubles the speed while held: exactly one octave up); page 2 — the master Filter (Cut · Reso ·
  FChar), Master Clock (Clock · ClkMd · ClkAt) and the ducking Pump (Pump · PmpRt).
- **External MIDI (Settings → MIDI):** three modes — **Off** (default) ignores external MIDI;
  **Keys** plays the loops from a MIDI keyboard (8-voice polyphony, each MIDI channel driving a
  loop chromatically through its full FX chain: channel 1 → loop 1, channel 2 → loop 2, … — but
  see [Known limitations](#known-limitations)); **Ctrl** hands external MIDI to the LaunchControl
  XL control surface (below). Off by default so Move tracks' MIDI-out can't trigger loops.
- **Settings (two pages):** page 1 is behaviour and I/O — arm threshold, overdub mode, **loop filter**
  (the 12 Master-filter voicings applied to every loop's low-pass), root note, **input channels**
  (`InCh`: Stereo · Left · Right · Sum — a mono synth on one jack records to both sides with `Left`),
  **input source** (see below), MIDI In and MIDI Out. Page 2 is **Output**, the master stage in
  signal order: Out · LoCut · HiCut · PWide · **Char** · gSat · Glue · Limit.
- **Character** (Output page) is thirteen hardware voicings ordered by grit — `Off · 962 · Air · SSL
  · Neve · Trident · Studer · API · Ampex · MPC · S950 · SP12 · Emu` — and it sets far more than an
  EQ curve. Each one chooses the **converter** (bit depth, sample-rate hold, µ-law companding on the
  MPC), the **saturator**'s curve and asymmetry (first-order ADAA, so the grit is the machine's, not
  aliasing), the **pole count** of LoCut/HiCut, how the loops are **read** (Hermite → linear →
  drop-sample), and it gives **Glue** and **Limit** the attack, release, knee and distortion of the
  dynamics unit that machine shipped beside — the 2254 behind a Neve, the 2500 (with its THRUST
  sidechain) behind an API, the SSL bus compressor.

| Char | Machine | What it sets |
|---|---|---|
| **Off** | — | transparent bypass |
| **962** | Studer 961/962 desk | near-flat, a whisper of transformer; 2-pole filters; no glue colour |
| **Air** | Focusrite ISA 110 | the air band: +3 dB at 15 kHz, 3-pole filters, clean and open |
| **SSL** | SSL 4000 bus | tight lows, present mids; 3-pole filters; the bus compressor's fast glue |
| **Neve** | Neve 1073 + 2254 | warm lows, silky top; 2254 glue with its diode-bridge distortion |
| **Trident** | Trident A-Range | broad mid lift, 2-pole; the slow, soft A-Range dynamics |
| **Studer** | Studer A800 tape | 60 Hz head bump, airy top; slow tape glue (30 ms / 600 ms) |
| **API** | API 550A + 2500 | forward 800 Hz, fast; 2500 glue with THRUST sidechain tilt |
| **Ampex** | Ampex ATR-102 | fat lows, the heaviest saturation; slowest glue, limiter at −1.7 dB |
| **MPC** | Akai MPC60 | 40 kHz hold, 12-bit **µ-law** companding, 18 kHz 2-pole reconstruction |
| **S950** | Akai S950 | 12-bit linear, 25 kHz, **6-pole** 10 kHz reconstruction filter; linear-interpolation reads |
| **SP12** | E-mu SP-1200 | 26.04 kHz, 12-bit, **no** reconstruction filter; **drop-sample** reads (repeats samples when slowed) |
| **Emu** | E-mu SP-12 | the same 26.04 kHz clock, darker fixed output (−4.5 dB at 5.5 kHz); drop-sample reads |

  Every figure, its source and whether it is documented, measured or chosen is in
  [docs/CHARACTER-RESEARCH.md](docs/CHARACTER-RESEARCH.md).
- **Record source (Settings p1 → InSrc):** `Line · Master · S1 · S2 · S3 · S4 · M1 · M2 · M3 · M4 · Self`.
  **Line** is the line/mic input (default). **Master** is the Move's whole master mix, minus Loopex's
  own output so it can never feed back. **S1–S4** are Schwung's own four mixer slots and **M1–M4** the
  four OG Move hardware tracks, each captured as an isolated stereo stem over **Ableton Link Audio**:
  the host publishes its slots on `/schwung-pub-audio` and reconstructs the Move tracks on
  `/schwung-link-in`, and Loopex reads the selected channel (needs the host's `link_audio_publish`, on
  by default). So a loop can sample any single Schwung or Move track on its own, not just the summed mix.
  **Self** is Loopex's own master output, one block (2.9 ms) old — every loop, the punch chain, both
  sends and the master stage — so you can resample the whole mix back into a new loop. Unlike
  **Master** it carries *no* feedback guard, because the regeneration is the point: the tape limiter
  at the end of the master chain keeps it saturating instead of diverging, and InGain is the control.
  If the host cannot provide a stem (they need host **1.4** with `link_audio_publish` on), Loopex records
  Line instead and says so on screen while Settings is open.
- **Undo** reverts the last overdub exactly (each overwritten sample is saved as it goes), else restores the last cleared loop.

### LaunchControl XL (external MIDI)
A Novation LaunchControl XL can drive all sixteen loops. Set **MIDI** (Settings p1) to **`Ctrl`** to
route external MIDI to the surface; **MIDI Out** (Settings p1 → `MidiO`) mirrors each loop's transport
back to the LCXL's LEDs, sent on **MIDI channel 2** so the notes never collide with the Move's own
channel-1 pads.

- **Track Focus buttons (row 1)** — record / play / pause per loop; **Track Control buttons (row 2)** —
  mute / unmute.
- Per column the **top knob is Loop Speed**, then Filter, then Pan, and the **fader is Volume**, for
  that loop.
- Two SysEx templates ship at the repo root — `BlackBox 1-8.syx` (loops 1–8) and `BlackBox 9-16.syx`
  (loops 9–16); set both to **MIDI channel 2** so the LED feedback lands.

---

## Controls

### Pads and steps
| Gesture | Action |
|---------|--------|
| **Tap** left pad | cycle Empty → Rec → Play ⇄ Pause (un-pause restarts from Start) |
| **Undo + pad** (playing/paused) | Overdub (tap again to stop) — or **hold the loop's step button** |
| **Hold** left pad (~1 s) | Clear the loop *(Undo restores it)* |
| **Shift + tap** | cycle playback speed (½× / 1× / 2×) |
| **Mute + tap** | quick-mute (playhead keeps running — returns in phase) |
| **Copy + pad, then pad** | clone a loop (source blinks, second pad receives it) |
| **Loop + pad** | cycle loop length 1× → ½× → ¼× → ⅛× |
| **Right pad** | punch-FX (momentary) · **Shift + pad** = latch · **Undo + pad** = reset params · **✕ + pad + step** = sequence |
| **Step** | select track (shows its waveform) · same step again = next loop page · **hold ~0.6 s = overdub that loop** (one hand) |
| **Track buttons 1–4** | menus: Input (Tape / EQ) · Perform · Send FX · Settings |

### Buttons and knobs
| Control | Action |
|---------|--------|
| **8 knobs** | selected-track params (or menu / punch params) |
| **Touch a knob** | full 8-knob page on screen (~5 s) |
| **Up / Down** | previous / next loop page (P1–P5) |
| **Jog wheel** | scrub the selected loop (audible, tape-style) · in P4 moves the touched head |
| **Capture** | Dynamic sampling menu · **hold** = sampler on/off (blinks while listening) |
| **✕ (Delete)** | tap = run/stop the FX sequencer · hold = pattern view + FX Seq page |
| **Left / Right** | tape stop / tape wind (held) |
| **Sample/Record** | Drift menu · **Shift + Sample** = threshold-arm (pad blinks red) · **+ jog** sets the threshold |
| **≡ (Menu)** | Sessions menu |
| **Mute / Copy / Loop** (held) | modifiers — lit while held |
| **Undo** | revert the last overdub, else restore the last-cleared loop |
| **Back** | close a menu, then exit |
| **Full exit** | **Shift + Volume + Jog-click** (a plain Back only *suspends*) |

### Screens
The main screen shows the 16-track strip, CPU and input level. Touching a knob or opening a
menu shows the **full 8-knob page**; pressing a step (or scrubbing) shows the loop's
**waveform with the active playheads riding over it**. Turning a loop's **Start** or **End** shows that
waveform with labelled **S** and **E** markers at the trim points. Both fall back after ~5 s.

---

## Signal chain

```
Record path: input -> tape model (13) -> tape drive -> input EQ -> HF loss (head physics)
             + head bump -> wow + flutter -> generations (the model's loss filter, once per pass)
             -> [loop buffers]

Per voice:   4 playheads (Hermite reads + rate-aware anti-imaging; the sampler Characters
             read linear / drop-sample) -> Seed slice re-order -> Scatter
             -> Pitch (Signalsmith Stretch, on its own thread) -> saturation -> wow/flutter
             -> DJ filter (+reso) -> tilt EQ -> Studer 962 EQ -> stability -> compressor
             -> amp env -> tape transport -> pan/vol -> sends A/B

Master:      sum of voices + MIDI-poly -> input monitor -> + Palette send returns
             -> master wow/flutter -> compressor -> Stumble -> [Clock + Filter, pre]
             -> punch-FX (5 in series) -> [Clock + Filter, post] -> Drift -> pump
             -> OUTPUT PAGE: gSat -> lo/hi cut -> Character -> Glue -> master out
             -> tape limiter -> TPDF dither -> output
```

---

## Build and install

Loopex is on the **Schwung Module Store** — most people can install it from the Schwung
Manager (search *Loopex*), no build needed. To build from source you need a host with
**Docker** (cross-compiles the ARM64 `dsp.so`) and **ssh/scp**, and an **Ableton Move**
running **Schwung** reachable over USB-C at `move.local` (or `172.16.254.1`):

```bash
cd overtake-shell
./scripts/build.sh      # Docker cross-compiles dsp.so (aarch64) + validates ui.js
./scripts/install.sh    # scp + atomic-rename install to move.local
```

Then on the Move: rescan modules and open **Loopex** from the Overtake list.
**If it was already loaded, full-exit first** (Shift + Volume + Jog-click) — `suspend_keeps_js`
otherwise resumes the old code.

`MOVE_HOST` overrides the target (`MOVE_HOST=ableton@172.16.254.1 ./scripts/install.sh`).

> **Deploy safety:** the installer stages to `/data/UserData` and moves files into place
> atomically — never write straight over a mapped `.so`, which crashes the audio process.

---

## Repository layout

```
overtake-shell/            <- the active module
  module.json              Overtake manifest (id/name/capabilities)
  src/
    loopex.c               engine: voices, playheads, punch-FX, Character, Drift, sessions, master
    palette_fx.c/.h        24-effect Palette engine + Dattorro Plate
    fx_clouds.cc           Clouds-based Space/Bloom (C++)
    pitch_shift.cc         Signalsmith Stretch wrapper (per-loop Pitch, run on the lpx-ps worker)
    warps_data.c           Warps wavetables (Fold/Shift)
    ui.js                  QuickJS Overtake UI (pads, knobs, screens, LEDs)
  include/plugin_api_v1.h  host API
  vendor/                  clouds_engine + signalsmith + signalsmith-stretch (third-party DSP)
  scripts/                 build.sh, install.sh, Dockerfile
docs/MANUAL.md             the full manual
docs/CHARACTER-RESEARCH.md every figure behind the 13 Character voicings, with sources
docs/CHANGELOG.md          release history
OVERTAKE-SDK.md            reverse-engineered Overtake SDK reference
design-spec.md             full design rationale
```

---

## Known limitations

- **MIDI-keyboard play reaches 4 loops, not 16.** In **Keys** mode each MIDI channel plays its
  own loop, and the module maps all sixteen (channel *n* → loop *n*). But a Schwung Overtake
  module only receives external MIDI through the host's **four forwarding slots**, so only four
  channels arrive at once — the other twelve have no slot to ride in on. To play loops 1–4, set
  four Schwung slots (or Move tracks) to receive channels 1–4 with **Forward = Auto**. This is a
  host delivery limit, not a module one (`MOVE_MIDI_SOURCE_FX_BROADCAST` is audio-FX-only, so a
  synth-role module can't tap it); lifting it to all 16 needs Schwung's `ext_midi_remap`
  passthrough enabled host-side, after which the existing mapping handles all sixteen unchanged.

---

## Changelog

Release history, with what changed and why: [docs/CHANGELOG.md](docs/CHANGELOG.md).

---

## Credits

Loopex stands on a lot of open work. Code that is vendored or ported is used under its own
license (see `overtake-shell/vendor/` and the file headers); the rest is inspiration.

### Code used
- **Mutable Instruments Clouds and Warps** — Émilie Gillet, MIT. The Palette texture and
  Space engines. https://github.com/pichenettes/eurorack
- **Signalsmith DSP library** — Geraint Luff, MIT. Filters, delays, STFT helpers.
  https://github.com/Signalsmith-Audio/dsp
- **Signalsmith Stretch 1.1.0** — Geraint Luff, MIT. The per-loop Pitch shifter.
  https://github.com/Signalsmith-Audio/signalsmith-stretch
- **Airwindows** — Chris Johnson, MIT. Ported or adapted: Spiral, Density, Mojo, Swell,
  Tremolo, Pressure4, ToTape6 (flutter), DeRez2 (Interference), IronOxide (global saturation).
  https://github.com/airwindows/airwindows · https://www.airwindows.com
- **Dattorro plate** — Jon Dattorro, *Effect Design Part 1: Reverberator and Other Filters*,
  JAES 45(9), 1997. The Plate send effect at the paper's delay lengths.
  https://ccrma.stanford.edu/~dattorro/EffectDesignPart1.pdf
- **Schwung** — Charles Vestal and contributors. The Overtake framework, and the knob-page
  widgets (arc knobs, enum squares, buttons, header, bank bar, footer pills) are ported
  verbatim from its `render_page_movy.mjs`. https://github.com/charlesvestal/schwung

### Sibling Move modules (same author)
- **Palette** — the 24-effect send engine Loopex embeds. https://github.com/filliformes/palette-move
- **Magnéto** — the tape-input stage, sessions, scrub and Tape page patterns. https://github.com/filliformes/magneto-move
- **Signal** — the Chop rhythm patterns. https://github.com/filliformes/signal-move
- **Punchfx** — the right-pad **Punch-in FX** bank: sixteen PO-33-style, pressure-sensitive
  momentary effects. Loopex's punch engine descends directly from it. https://github.com/filliformes/punchfx-move
- **Fizzik** — the **12 loop/master filter voicings** (Clean · SEM · MS-20 · Steiner · three
  ladders · Prophet · Oberheim · Diode · K35 · Vintage): original analog-voiced designs built on
  a trapezoidal SVF + zero-delay-feedback transistor ladder (Zavalishin/Cytomic VA-filter math,
  RBJ cookbook); the MS-20 voicing was modelled from original Korg MS-20 schematics while working
  on Aphex. https://github.com/filliformes/fizzik-move
- **Structor** — several punch-effect ideas. https://github.com/filliformes/structor-move

### Other software instruments (same author)
- **Res** and **Essaim** — the Quartz and Prism reverbs: one dual-mode 8-line Hadamard FDN whose
  two feedback colourings fill the roles of Ableton's `abl.dsp.quartz~` and `abl.dsp.prism~` Max
  objects.
- **Phasma** — the Veil reverb: a modulated Householder FDN (Signalsmith-style tank with
  CloudSeed-style modulated diffusers) carrying Prism's band-split colouring.

### Other Schwung modules
- **Performance FX** — Charles Vestal. The framework author's pressure punch-in FX module,
  the pattern the Punch-in bank grows from (Loopex carries 16 on the right pads).
  https://github.com/charlesvestal/schwung-performance-fx
- **Smack** — Tim Cox. The Seed slice re-order is modelled on its Seed parameter.
  https://github.com/timncox/schwung-smack
- **Forgetful** — Idlir Fida. The Stumble perform gesture. https://github.com/kliegsablaze/forgetful

### Design inspiration
- **Polyend MESS** — the FX sequencer: per-step locks, play chance, extensions, gate and swing.
  https://polyend.com/mess/
- **1010music Blackbox** — sixteen free-running pads. https://1010music.com/product/blackbox
- **Kinotone Ribbons** — tape character as an instrument. https://kinotone.com/ribbons
- **Puremagnetik LAPS** — layered asynchronous loops. https://puremagnetik.com
- **Chase Bliss Blooper, Mood MK2, Generation Loss MK2** — Stability, the old Clock's
  degradation, Disintegration overdub, Generations. https://www.chasebliss.com
- **Hologram Microcosm** — the grain and glide punch families. https://hologramelectronics.com/microcosm
- **Character voicings** — every figure behind the thirteen Char models, with sources and an
  explicit mark on what is documented versus chosen: [docs/CHARACTER-RESEARCH.md](docs/CHARACTER-RESEARCH.md).
- **Studer 961/962** — the per-loop channel EQ, modelled on the console's *Fächerentzerrer* (fan
  equaliser) in the 1.960.221 input unit. Bass and Treble corners and slopes are a least-squares
  fit to the response curves printed in the service manual (§1.7.2 / D 3/3), digitised off the
  plot; Mid follows the published Q = 1, 150 Hz–7 kHz, ±11 dB presence spec.
- **Soma Laboratory COSMOS** — the Drift memory: prime-length shifting delay lines that recombine
  endlessly into an ever-evolving ambient layer. https://somasynths.com/cosmos/
- **norns loopers** — wrms, concrète, cranes, oooooo, otis, reels, ndls, samsara, mlre, nydl,
  giro: multiple playheads, threshold arm, loop multiples, jog scrub. https://norns.community

## License

GPL-3.0
