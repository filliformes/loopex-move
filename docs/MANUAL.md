# Loopex Manual

*16-track stereo tape looper for Ableton Move (Schwung Overtake module) — v0.9.2*

This is the full reference. The [README](../README.md) is the short tour.

---

## 1. Layout

```
 ┌────────────────────────── screen ──────────────────────────┐   [K1] [K2] [K3] [K4] [K5] [K6] [K7] [K8]
 │ header · knob page / waveform / overview · footer hints     │
 └─────────────────────────────────────────────────────────────┘
 [Step 1 … Step 16]                       ← select a loop / FX-sequencer pattern
 [Track 1] [Track 2] [Track 3] [Track 4]  ← menus: Input · Perform · Send FX · Settings
 ┌ left 4×4 pads ┐  ┌ right 4×4 pads ┐
 │  16 loops     │  │  16 punch FX   │
 └───────────────┘  └────────────────┘
```

- **Left pads** are the sixteen loops. Their colour shows the state (empty · recording · playing · paused · overdubbing) tinted by speed (blue ½×, green 1×, yellow 2×).
- **Right pads** are the sixteen punch-in effects, coloured by family (blue Loops, pink Grains, purple Pitch, yellow Time, red PalFX).
- **Steps** select the loop whose knobs you are editing, and double as the FX-sequencer pattern while ✕ is held.
- **Knobs 1–8** edit the page on screen. Touching a knob shows the full page and inverts the header with the parameter's full name and value while you hold it.

The screen has three views: the **overview** (track strip, CPU, input level, footer hints), the **knob page** (Schwung's arc knobs, enum squares, buttons and big numbers), and the **waveform** of the selected loop with its playheads. Knob pages and the waveform fall back to the overview after ten seconds without input.

---

## 2. Loops

### 2.1 Recording and playback

| Gesture | Action |
|---|---|
| Tap an empty pad | start recording (up to 45 s) |
| Tap while recording | close the loop and play |
| Tap while playing | pause · tap again to restart from Start (click-free) |
| **Undo + pad** while playing or paused | overdub · tap again to stop overdubbing |
| **Hold the loop's step button** (~0.6 s) | overdub, one-handed · hold again to stop |
| Hold a pad (~1 s) | clear the loop (Undo restores it) |
| Shift + tap | cycle playback speed ½× · 1× · 2× |
| Mute + tap | quick mute; the playhead keeps running so the loop returns in phase |
| Copy + pad, then a second pad | clone the first loop into the second (audio and settings, done on the worker) |
| Loop + pad | cycle the loop length 1× · ½× · ¼× · ⅛× |
| Shift + Sample, then a pad | threshold-armed record: the pad blinks red and recording starts when the input crosses the threshold (Shift + Sample + jog sets it) |

Loops are free-running: they do not need to share a length or a downbeat.

### 2.2 Overdub modes (Settings → ODub)

- **Replace** — new audio replaces the old.
- **Multiply** — endless layering; older passes decay each cycle. **This is the default.**
- **Disintegration** — the loop's own effects are baked in on every pass, so it slowly falls apart.

### 2.3 Undo

**Undo** reverts the last overdub exactly. While an overdub runs, every sample it overwrites is saved the moment it is replaced, so the restore is sample-accurate and costs nothing on the audio thread; the restore itself runs on the worker. If no overdub is pending, Undo restores the last cleared loop instead.

### 2.4 Selecting and pages

Press a **step** to select that loop: the screen shows its waveform with the active playheads riding over it. Press the same step again to cycle through its five knob pages. **Hold** a step for about half a second and that loop goes into **overdub** — the one-handed alternative to Undo + pad; hold again to stop. **Up** and **Down** move between pages from anywhere, including the waveform view.

| Page | K1 | K2 | K3 | K4 | K5 | K6 | K7 | K8 |
|---|---|---|---|---|---|---|---|---|
| **P1 Loop** | Speed | Filter | Pan | Volume | Start | End | Reverse | Send A |
| **P2 Texture** | Pitch | Reso | Sat | Comp | Wow/Flutter | Scatter | Seed | Send B |
| **P3 Tone** | Bass | Mid Freq | Mid Gain | Treble | Tilt | Attack | Decay | Heads ▸ |
| **P4 Playheads** | H1 mode | H1 speed | H2 mode | H2 speed | H3 mode | H3 speed | H4 mode | H4 speed |
| **P5 HeadMix** | H1 Vol | H1 Pan | H2 Vol | H2 Pan | H3 Vol | H3 Pan | H4 Vol | H4 Pan |

- **Speed** — playback rate, ±2 octaves in 0.1-semitone steps. Pitch and tempo move together, like tape.
- **Pitch** — an independent pitch shift, −24 to +24 semitones, that leaves the tempo alone. It is a Signalsmith Stretch phase-vocoder shifter running on its own worker thread, off the audio callback, so a pitched loop costs the engine almost nothing; its latency (61 ms including the hand-off) is cancelled by nudging the loop's playheads, so a shifted loop stays in time. At exactly 0 it is fully bypassed.
- **Filter + Reso** — a DJ-style filter: left of centre low-pass, right of centre high-pass, resonance on Reso. Smoothed over 10 ms.
- **Start / End** — loop window. End is a length from Start.
- **Sat** — per-loop tape saturation with unity makeup.
- **Comp** — per-loop compressor; makeup gain fades in with the knob.
- **Wow/Flutter** — per-loop tape instability.
- **Scatter** — random slice jumps, crossfaded.
- **Seed** — a seeded slice re-order (2/4/8/16 slices, some reversed). The knob *is* the seed: every position is a different, repeatable mangle.
- **Bass / Mid Freq / Mid Gain / Treble** — the Studer 961/962 channel EQ, a *fan equaliser*:
  every setting crosses 0 dB at a ~1 kHz pivot and slopes continuously outward, with no shelf
  plateau in the audio band. Bass reaches ±15 dB at 20 Hz, Treble ±15 dB at 20 kHz — those are
  the gains at the edges of the band, which is why the controls stay gentle and musical in the
  middle. Mid is a ±11 dB presence peak (Q = 1) sweepable 150 Hz–7 kHz.
  **Tilt** tips the whole spectrum.
- **Attack / Decay** — a per-loop amplitude envelope (3 ms – 3 s / 5 s) used on trigger, mute, pause and stop.
- **Heads ▸** — jumps to the Playheads page.

### 2.5 Playheads

Every loop can be read by four heads at once. Modes are **Off · Fwd · Bwd · Ping · Jump**; speeds run 0.25× to 4× in 0.1-semitone steps. **Jump** plays forward but leaps to a random position at random intervals, each leap crossfaded. Head 1 is the main head (Scatter, Seed and scrub drive it). Turning a head on restarts it at the loop start. Heads sum with 1/√n normalisation. Any head that moves position (jog, pitch-shift nudge, scatter, a Jump leap) does so through a short crossfade.

In the Playheads page, **touch a head's knob and the jog wheel moves that head** along the waveform. Everywhere else the jog **scrubs** the selected loop, tape-style: the head travels for a moment and you hear it, even on a paused loop.

### 2.6 HeadMix

The fifth page gives every playhead its own **Volume** and **Pan** (a balance: centre is unity, full pan mutes the opposite side). **H1 Vol/Pan are the same controls as loop page 1** — head 1 is the main head, so its level and position *are* the loop's. Heads 2–4 get independent gain and pan, applied per head before the heads sum, defaulting to unity/centre so a loop sounds unchanged until you move them. All of it saves with the session and travels when you clone a slot.

---

## 3. Punch-in FX (right pads)

Sixteen momentary effects on the master, reading a 2-second capture ring. Hold a pad to apply; up to **five** can run in series. Each pad has four parameters on knobs 5–8 while it is held, and **pad pressure** drives a per-effect expression shown in the footer.

| Row | Pads | Knobs 5–8 | Pressure |
|---|---|---|---|
| **Loops** | Loop12 · Loop16 · LoopSh · **Chop** | Rate · Pitch · Tone · Mix (Chop: Rate · **Pattern** · Tone · Mix) | subdivides the loop continuously (Chop: rate ×2) |
| **Grains** | Haze · Mosaic · Smear · Strum | Size/Grid/Rate · Pitch/Dir · Density/Var/Tone · Mix | density · grid ×2 · density · faster + wider |
| **Pitch** | Oct− · Oct+ · Glide · Shimmer | Fine/Len/Regen · Pitch/Glide · Tone · Mix | mix · mix · glide · regen |
| **Time** | Stretch · Freeze · Reverse · **PalFX** | Stretch/Frz/Len · Pitch · Grain/Tone · Mix (PalFX: **FX · Amount · Macro · Drift**) | freeze · freeze · shorter · amount |

- **Chop** repeats the last transient on one of 32 rhythmic patterns (Morse bursts, Ikeda burst/silence, pairs, straight eighths, syncopated) chosen with knob 6.
- **PalFX** runs one Palette effect (all 26, default Space) as a punch. It is block-processed, so its wet is one block late, like the send buses.
- Slice effects auto-pan in time with their rate. Every slot loudness-matches its wet to the dry it replaces. Loops, Reverse, Glide, Chop and a frozen Stretch freeze their capture ring while held, so a loop held longer than two seconds is never overwritten under the head.

| Gesture | Action |
|---|---|
| Hold pad | apply |
| Shift + pad | latch (hands-free); Shift + pad again unlatches |
| Shift while holding a pad | latch it exactly as it is, pressure included |
| Undo + pad | reset the pad's four knobs to their defaults |
| ✕ + pad(s) + step | write the pads into the FX sequencer (section 5) |

Knobs 5–8 only follow a pad while it is physically held; latched pads keep running but give the knobs back to the page.

---

## 4. Menus (Track buttons, Capture, Sample, Menu)

| Button | Menu | K1 | K2 | K3 | K4 | K5 | K6 | K7 | K8 |
|---|---|---|---|---|---|---|---|---|---|
| Track 1 | **Input Tape** | Monitor | Tape | Trim | Drive | Wow | Flutter | Hiss | Generations |
| Track 1 (page 2) | **Input EQ** | LoCut | HF | **LowF** | Low | Mid | MidF | High | HiF |
| Track 2 | **Perform** | Stumble Mix | Stumble Step | Stumble Odds | Stumble Size | Stumble Reach | Stumble Kind | Jump | Scan |
| Track 2 (page 2) | **Perform 2** | Filter Cut | Reso | FChar (filter model) | Clock | ClkMd (Music / Free) | ClkAt (pre / post punch) | Pump | Pump Rate |
| Track 3 | **Send FX** | A FX | A Amount | A Macro | A Drift | B FX | B Amount | B Macro | B Drift |
| Track 4 | **Settings** | Arm Threshold | ODub mode | **LpFlt** (loop filter) | Root | **InCh** (input channels) | **InSrc** (input source) | MIDI In | MIDI Out |
| Track 4 (page 2) | **Output** | Out | LoCut (20–1000 Hz) | HiCut | PWide (punch width) | Char | gSat (to 2.0) | Glue | Limit |
| Capture | **Dynamic** | Dyn (mode) | Sense | Size | Spread | Error | Sustain | Rnd Pad | Rnd All |
| Sample | **Drift** | Drift | Rate | Size | FBk | Supr | Blur | Damp | Mix |
| ≡ (Menu) | **Sessions** | Slot | Save | Load | Del | **Clear** | **Reset** | | |
| ✕ (held) | **FX Seq** | Run | Speed | Length | Chance | Gate | Swing | Direction | Clear |

Press the same button again, or **Back**, to close a menu. Enums step once per four detents so a fast turn does not race through the list.

- **Input** (track button 1, two pages) — **Input Tape** is the machine: Monitor level, one of 13 models (`Tapeless · Clean · Cass1 · Cass2 · VHS1 · VHS2 · Reel15 · Reel7 · Reel3 · 4trk · Porta · Dub · Warp`, default Clean), Trim, Drive, Wow, Flutter, Hiss and Generations. Each model's high-frequency loss is derived from its head gap and tape speed, with a speed-scaled head bump; VHS1 is the Hi-Fi track (FM carrier + companding NR), VHS2 the linear edge track; Generations re-applies the machine's own loss filter once per pass. Tapeless bypasses the machine (no hiss floor, no head loss or bump) but every knob still works; Drive pushes up to +24 dB into the curve on every model. **Input EQ** (page 2) blends the Studer 962 presence band with the Tascam 424 MkII/MkIII channel EQ (owner's manual p.11/36/45): a 20–800 Hz low cut, the 2–20 kHz tape rolloff (HF: turn *down*), ±15 dB 2nd-order shelves with a sweepable low corner (LowF 40–400 Hz, default 100 Hz, the 424's point) and a 3–15 kHz high corner (default 10 kHz), and a ±12 dB peak sweepable 150 Hz–7 kHz. Every sweep is log. It shapes what gets recorded.
- **Dynamic** (Capture button) — the input plays the sampler. Set **Dyn** to a mode, or **hold Capture** to toggle it (the button blinks while listening), and every phrase you play is captured into a pad by itself: an ordinary recording that gets the tape stage, the loop pages, the sends, and whose LED goes red then green like any take. What was on the pad is *replaced*, never stacked; **Spread** is how many pads the sampler rotates through from the selected one (1 = the same pad every time; 16 = a rolling memory of the last sixteen phrases). Muted pads are skipped, and a pad that already holds a loop is never overwritten until you say so — when the range is full the menu opens with `ALL PADS FULL: OVERWRITE?` (K8 yes, K5 or Back turns Dynamic off; it asks again next time you turn it on). Modes: **Level** (a sound above the threshold), **Onset** (a transient, however loud the sustain under it), **Phrase** (starts on sound, stops at the next gap), **Clock** (every Size on Move's tempo, no pre-roll). **Sense** is the mode's threshold (−60 → −6 dBFS); **Size** is 1/16 … 8 bars at Move's tempo or **Free** (until 200 ms of quiet — the default); **Error** is how much of each new capture is randomised; **Sustain** is how long a capture plays before it pauses itself and its Decay fades it (1 → 60 s, top = forever). A 100 ms pre-roll keeps the transient that triggered a capture. **Rnd Pad / Rnd All** randomise the loop pages of the selected pad / all sixteen, without confirmation, by musical rules: Speed and Pitch move only as complementary musical intervals (the heard transposition and the tempo ratio are both octaves, fifths or major/minor thirds, never more than two octaves; a third of the time nothing moves); playhead speeds are never touched; heads 2–4 are switched on or off and only an active one gets a random pan; the loop's Volume is left alone; sends, Comp and Sat never go past 60 %.
- **Send FX** — two Palette buses (29 effects: Drive, Sweeten, Fuzz, Howl, Fold, Swell, Doubler, Vibrato, Phaser, Tremolo, Pitch, Shift, Cascade, Reels, Collage, Reverse, Space, Bloom, Filter, Squash, Cassette, Broken, Interference, Halo, Plate, Quartz, Prism, Veil), each with Amount, Macro and Drift. Every effect is loudness-matched to the dry signal it replaces, and switching effects **morphs** — the outgoing effect fades out as the incoming one fades in — so sweeping through the list never clicks. (The swap itself happens on the worker.)

  The last four are full reverbs, all 100% wet (they sit on a send):

  | Effect | Tank | Amount | Macro | Drift |
  |---|---|---|---|---|
  | **Plate** | Dattorro plate at the paper's delay lengths | decay | pre-delay + darkening | damping |
  | **Quartz** | 8-line Hadamard FDN, 4 input diffusers, HF one-pole + LF shelf damping | room → hall | dark → bright | modulation + pre-delay |
  | **Prism** | the same tank with per-band RT60: lows, mids and highs decay separately | decay length | tilt: lows ring ⇄ highs shimmer | crossover + modulation |
  | **Veil** | Householder FDN with LFO-modulated diffusers, long in-loop-damped lines | size + tail | dark → bright | movement + band colour |

  Quartz and Prism are ported from Res and Essaim, where they stand in for Ableton's
  `abl.dsp.quartz~` and `abl.dsp.prism~`; Veil is Phasma's tank, which is what makes it
  lush rather than metallic. Size changes glide, so turning Amount morphs the room
  tape-style instead of clicking.
- **Perform** — page 1: Stumble (a probabilistic step glitcher), plus **Jump** (crossfaded random jump on every playing loop) and **Scan** (doubles the playback speed while held — exactly one octave up) as buttons. Page 2: the **master Filter** (Cut · Reso · FChar, the twelve Fizzik voicings), the **Master Clock** (a Mood-style pitch/rate warp; ClkMd snaps to intervals or slides free, ClkAt places it before or after the punch bank) and the tempo-synced **Pump** (depth · rate).
- **Output** (Settings page 2) — the master stage in signal order: Out · LoCut · HiCut · PWide · **Char** · gSat · Glue · Limit. **Char** is thirteen hardware voicings ordered by grit, and it sets more than an EQ curve: each one chooses the converter (bit depth, sample-rate hold, µ-law companding on the MPC), the saturator's curve and asymmetry (first-order anti-aliased, so the grit is the machine's rather than aliasing), the pole count of LoCut/HiCut (2, 3 or 6), how the loops are read (Hermite → linear → drop-sample), and gives **Glue** and **Limit** the attack, release, knee and distortion of the dynamics unit that machine shipped beside.

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

  Every figure, its source, and whether it is documented, measured or chosen is recorded in [CHARACTER-RESEARCH.md](CHARACTER-RESEARCH.md).
- **Drift** (Sample button) — a global drifting-delay memory in the spirit of Soma COSMOS. Four coprime-length delay lines, each read at a slowly drifting tap, feed back through a matrix that morphs from self-feedback to a normalised Hadamard cross-mix. The loop mix feeds it, the memory recirculates, and because the line lengths are coprime and each has its own asynchronous LFO, the recombination never lands on an exact repeat. It sits in the master chain just before the pump, so the ambient layer picks up the character EQ, glue and limiter. Feedback is capped below unity, so the tail always fades (up to a few minutes at maximum), and a **silence bleed** clears an abandoned tail after about eight seconds with no input. Knobs: **Drift** (how much loop mix is fed in) · **Rate** (tap-drift speed) · **Size** (tap length, shimmer to long hall) · **FBk** (memory sustain, below unity fades, near unity holds) · **Supr** (loud new input erases old memory: play over to replace) · **Blur** (self-feedback → full cross-mix) · **Damp** (high-frequency damping of the tail) · **Mix** (wet level into the master). Drift and Mix start at zero, so it is silent until dialled in; it saves with the session.
- **InCh** (input channels, Settings page 1) — `Stereo · Left · Right · Sum`. A mono synth on a single jack arrives on one side only; **Left** (or **Right**) copies that side to both, **Sum** mixes the two. Default Stereo.
- **InSrc** (input source, Settings page 1) — what each new recording samples:
  `Line · Master · S1 · S2 · S3 · S4 · M1 · M2 · M3 · M4 · Self`. **Line** is the line/mic input (default).
  **Master** is the Move's whole master mix, minus Loopex's own output so the master can never feed back.
  **S1–S4** are Schwung's own four mixer slots and **M1–M4** the four OG Move hardware tracks, each
  captured as its own isolated stereo stem over **Ableton Link Audio** — the host publishes its slots on
  `/schwung-pub-audio` and reconstructs the Move tracks on `/schwung-link-in`, and Loopex records the
  selected channel. This lets a loop sample any single Schwung or Move track in isolation, not just the
  summed mix. It needs the host's `link_audio_publish` (on by default).
  **Self** is Loopex's own master output, one block (2.9 ms) old — every loop, the punch chain, both
  sends and the master stage — so you can resample the entire mix back into a new loop (bounce,
  layer, regenerate). Unlike **Master** it has *no* feedback guard, because the regeneration is the
  point: the tape limiter at the end of the master chain makes it saturate rather than run away, and
  **InGain** is how you control how hard it builds. If the host cannot provide a stem (they need host **1.4** with `link_audio_publish` on), Loopex records Line instead and says so on screen while Settings is open.
- **MIDI In** — off by default so Move's track MIDI cannot trigger loops. On, an external keyboard plays the selected loop chromatically with 8-voice polyphony, and a LaunchControl XL drives all sixteen loops (section 9).
- **MIDI Out** — mirrors each loop's transport state to the LaunchControl XL's LEDs (section 9).

---

## 5. FX sequencer (✕ / Delete)

One shared 16-step pattern of punch pads, in the spirit of the Polyend MESS.

| Gesture | Action |
|---|---|
| Tap ✕ | run / stop (the LED is bright while running) |
| Hold ✕ | pattern view on the steps: orange = step with FX, dim orange = extension, white = playhead; the FX Seq page opens. Turn a knob and the page stays open after release |
| ✕ + pad(s) + step | write up to five pads into the step. Each pad's four knobs and current pressure are locked into the step, so editing the pad later leaves the step alone. Pads pressed while ✕ is held are silent selections |
| ✕ + step (no pads) | clear the step |
| ✕ + hold a step + tap a later step | the steps between become extensions that hold the first step's effects |
| ✕ + hold a step + K4 | that step's Play Chance |
| ✕ + hold a step + K5–K8 | the locks of the step's first effect |
| Play | restarts the pattern from step 1 |

Page: **Run** · **Speed** (1/32 · 1/16 · 1/8T · 1/8 · 1/4 · 1/2 · 1 beat) · **Length** (1–16) · **Chance** (Always · 10–90% · Like Last · Play 1 Skip 1 · Play 2 Skip 1 · Skip 1 Play 1; the default for new steps when no step is held) · **Gate** (10% – Hold) · **Swing** (50–75%) · **Direction** (Fwd · Bwd · Ping · Rand) · **Clear** (asks first).

The clock runs from the Move tempo. A step retriggers its effects the way a finger would, through a short fade. A pad you are holding by hand is never cut by the sequencer. Gate below Hold releases the effects part-way through the step unless the next step is an extension. The pattern and its locks save with sessions.

---

## 6. Tape transport (◀ ▶)

Hold **Left** and the whole master brakes to a stop in about three seconds; hold **Right** and it winds up to a tone. Release and it eases back to 1× in a third of a second. The glide is linear in semitones and the last octaves of a stop fade to silence, so it never sits on a hum.

---

## 7. Sessions (≡ Menu button)

Sixty-four slots. **Slot** browses (one slot per four detents), **Save** writes, **Load** reads, **Del** (K4) erases the slot, **Clear** (K5) wipes all sixteen loops and resets every loop's settings to factory — after a confirmation. The global pages (Input, Output, Perform, Drift, Sends) are left as they are: it is the loops, not the rig. **Reset** (K6) takes every *setting* of the selected loop back to factory — speed, filter, EQ, sends, playheads, HeadMix, all of it — after a confirmation. The audio stays and keeps playing from where it was; hold the pad if you want that cleared too. Saving over a used slot, or deleting one, asks first (K8 = yes, K5 = no, Back cancels). Each slot is named by date and time (`Sep 14 21:30` in the footer); the header shows which session is loaded, or **New**. A successful save shows a burst.

A session holds every setting, the punch pad values, the FX-sequencer pattern and all recorded audio. The audio is stored as plain **16-bit stereo 44.1 kHz WAV** files — `sessionNN/sNN_loop01.wav` … `sNN_loop16.wav`, one per loop — so you can copy them off the Move (`scp ableton@move.local:/data/UserData/UserLibrary/Loopex/session03/*.wav .`) and use them anywhere — or browse them in the Schwung Manager (`move.local:7700` → Files → `data/UserData/UserLibrary/Loopex`). Sessions saved before 0.9.2 are moved there on first launch and still load. Disk work runs on a worker thread pinned to cores 0–2, never on the audio callback. Files live in `/data/UserData/UserLibrary/Loopex/` and survive reinstalls.

---

## 8. Signal chain

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

## 9. External MIDI — LaunchControl XL

A Novation LaunchControl XL can play all sixteen loops from hardware. **MIDI In** (Settings page 1 → **MIDI**) turns external control on; **MIDI Out** (Settings page 1 → **MidiO**) mirrors each loop's transport state back to the LCXL's LEDs. The LED feedback is sent on **MIDI channel 2** so its Track Focus/Control notes never collide with the Move's own channel-1 pads — set the two templates to channel 2 to receive it.

| LCXL control | Action |
|---|---|
| **Track Focus buttons (row 1)** | record / play / pause the loop (per column) |
| **Track Control buttons (row 2)** | mute / unmute the loop |
| **Top knob** | Loop Speed |
| **Middle / bottom knob** | Filter · Pan |
| **Fader** | Volume |

- LED colours mirror the loop: red while recording, green while playing/overdubbing, amber while paused; the Track Control LED is red when the loop is muted.
- Two SysEx templates ship at the repo root — `BlackBox 1-8.syx` (loops 1–8) and `BlackBox 9-16.syx` (loops 9–16). Load them onto the LCXL and set both to **MIDI channel 2**. Loopex accepts both template ranges at once, so it works whichever is active.

---

## 10. Shortcut sheet

| Keys | Action |
|---|---|
| Pad tap / Undo+pad / hold | rec-play-pause · overdub · clear |
| Shift + pad | speed ½× 1× 2× |
| Mute + pad · Copy + pad, pad · Loop + pad | quick mute · clone · loop length |
| Shift + Sample (+ jog) | threshold-arm (+ threshold) |
| Right pad · Shift + right pad · Shift while held · Undo + right pad | punch · latch · latch as-is · reset |
| Step · same step again · hold step · Up / Down | select loop (waveform) · next page · overdub · prev/next page |
| Jog | scrub · in P4: move the touched head |
| Track 1–4 · Capture · Sample · ≡ | Input · Perform · Send FX · Settings · Dynamic · Drift · Sessions |
| Capture (held ~0.6 s) | Dynamic sampler on / off (blinks while listening) |
| ✕ tap · ✕ hold · ✕ + pads + step · ✕ + step · ✕ + step + step | seq run/stop · pattern view · write · clear · extend |
| ◀ / ▶ (held) | tape stop · tape wind |
| Undo | revert last overdub, else restore last clear |
| Back | close menu / popup, then leave |
| Shift + Back (or Shift + Volume + Jog-click) | full exit (a plain Back only suspends) |
