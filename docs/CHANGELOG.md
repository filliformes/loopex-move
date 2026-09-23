# Changelog

## v0.9.2 — unreleased

### Tape Wear — the Disintegration Loops, per loop
- **New knob: Wear** (Tone page, K8). Your loops can now wear out the way William Basinski's ageing
  tape loops did in *The Disintegration Loops*, where the oxide flaked off a little more on every pass
  over the head. Every time a playhead crosses a spot on the loop, that spot loses some oxide: the
  treble goes first (the physics of a head lifted off the tape by loose flakes), then the level, until
  the spot drops out entirely. Damage starts at a few weak places, grows where it started and speeds
  up near the end — so the music breaks into the *same* holes every lap and stays recognisable while
  it dies. It is truly stereo — like the two tracks side by side on real tape, each has its own
  damage, mostly shared, partly its own, the edge track wearing a little faster — so a hole often
  opens on one side first and the stereo image wanders as the loop dies. Every active playhead
  wears the tape, so four heads rot it faster.
- **The knob is a speed, not an amount.** At the top an 8-second loop collapses in about a minute; in
  the middle, about twenty; near the bottom it takes hours. At 0 the tape stops wearing — the damage
  stays exactly where it is. Only **Reset** (Shift + Undo + pad, or Sessions → Reset) heals it.
- **Nothing is destroyed.** The recording and the saved WAV stay clean; the damage is a map laid over
  it, saved with the session (a half-dead loop reopens half-dead), copied when you clone a loop, and
  undone with everything else. A new recording on a pad starts on fresh tape.

**What behaves differently from before**
- **The Tone page's K8 is Wear, not Heads ▸.** The Playheads page is still one press of **Down** (or a
  second tap on the loop's step) away.
- **Wear is not the Disintegration overdub mode.** That mode (Settings → ODub) still re-applies the
  loop's own effects into the audio once per *overdub pass* and changes the recording itself. Wear
  works on any recorded loop, while it simply plays, on its own clock, and never touches the audio.
- **Reset now also heals worn tape**, and Undo of a reset / randomise / clear brings the damage back
  with the rest.

### Dynamic sampling (Capture button)
- **The input plays the sampler.** Four modes — **Level**, **Onset**, **Phrase**, **Clock** — capture
  every phrase you play into a pad by itself, as an ordinary recording (tape stage, loop pages, sends,
  LEDs). What was on the pad is replaced, never stacked (Onward); **Spread** rotates through 1–16
  pads (Continua's *dimension*, sixteen layers wide); muted pads are skipped; a pad with audio is never
  overwritten until you answer `CONTINUE DYNAMIC LOOPING AND OVERWRITE PADS?` (K6 yes, K5 no — it takes over any view). **Sense**, **Size** (tempo divisions or
  Free), **Error** (each capture randomised by that much), **Sustain** (a capture pauses itself after
  1–60 s, or never). 100 ms pre-roll keeps the triggering transient. Long-press Capture toggles it;
  the LED blinks while listening. Pitch- and novelty-triggered modes to follow.
- **Rnd Pad / Rnd All** randomise the loop pages by musical rules (each FX stage engaged with a 50 %
  chance, otherwise off — everything on at once on sixteen loops was neither musical nor affordable): Speed and Pitch only as complementary
  musical intervals, playhead speeds untouched, heads 2–4 on/off with pan only when active, Volume
  untouched, the Start/End window never under half the loop, sends / Comp / Sat capped at 60 %.

### Input
- **Generations** now re-applies the machine's head bump per pass and adds wow per dub, alongside the
  cascaded loss filter (a static "as if dubbed N times" setting, up to four).
- **Input FX and Input Tape merged** under track button 1 as *Input Tape* / *Input EQ*; Capture freed.
- **Input EQ blended** from the Studer 962 presence band and the Tascam 424 MkII/MkIII channel EQ
  (owner's manual): ±15 dB shelves with a sweepable low corner (**LowF** 40–400 Hz, default 100) and a
  3–15 kHz high corner (default 10 kHz), ±12 dB mid 150 Hz–7 kHz; every sweep log, LoCut 20–800 Hz log.

### Undo
- **Sixteen levels.** Undo walks back through the last sixteen gestures, newest first: overdubs,
  clears, arms, Dynamic overwrites, randomise, reset. A gesture that touched several pads (Rnd All,
  Clear) comes back in one press. Audio undo swaps buffers rather than copying, so it is instant.
- **Undo + pad no longer overdubs** (that is the step hold); **Shift + Undo + pad** resets that loop's
  settings, the same as Sessions → Reset.

### Gestures
- **Start and End are precise and fluid.** A slow turn moves 2 ms of audio per detent whatever the
  loop's length (it was a fixed 0.6 % of the loop — 270 ms per click on a 45 s loop); turning faster
  accelerates smoothly, so one quick twist still crosses the whole loop. The position is kept to the
  sample through saves and reloads.
- **Hold a loop's step button (~0.6 s) to overdub it** — one-handed, no menu, no mode. Hold again
  to stop. Undo + pad still works. A short tap selects as before; on the selected loop the tap
  flips to the next page on release. (Thanks Hannes.)
- **Speed and length readouts stay on screen ~2.5 s** (`T3 speed 1/2x`, `T3 length 1/4x`) instead of
  vanishing under the footer hint. (Thanks djd_oz.)

Changed in 0.9.0, never announced:
- **Overdub is Undo + pad**, not a double-tap. The double-tap forced every pad press to wait and see
  whether a second one was coming, so a pad did not stop the moment you hit it — and overdubs
  started by accident. Undo + pad is instant and never misfires.
- **Tapping a recorded pad**: first tap plays, second tap **stops** it, third tap starts it again
  **from the Start point**, re-phasing all four playheads.

### Sessions
- **Sessions live in the Move's UserLibrary and save loops as WAV.** Sessions moved from
  `/data/UserData/schwung/loopex-sessions` to `/data/UserData/UserLibrary/Loopex` — where the
  Schwung Manager's file browser shows them, next to Magnéto's recordings — and each loop is a plain
  `sessionNN/sNN_loop01.wav` … `sNN_loop16.wav` (16-bit stereo 44.1 kHz; two-digit numbers so
  the Manager's file browser sorts them). Existing sessions are moved on first launch (an atomic
  rename, nothing copied) and every old `tNN.raw` is converted to WAV, then removed. (Thanks Hannes.)
- **Clear** on the Sessions page (K5): wipes all sixteen loops and resets their settings after a
  confirmation (global pages untouched), header back to *New*. K5 is also *NO* inside the popup, so a second turn cancels
  rather than wipes. (Thanks Hannes.)
- **Reset** on the Sessions page (K6): every setting of the selected loop back to factory — the audio
  stays and keeps playing — after a confirmation. **Reset All** (K7) does it for all sixteen. **Clear** resets the loops' settings too. The defaults are the very same function
  `create_instance` uses, so "reset" means exactly "as new". (Thanks Hannes.)

## v0.9.1 — 2026-09-20

Fixes from the first day of user feedback on 0.8.7 / 0.9.0.

- **Input Tape knobs work on every tape model.** They were gated on the model, which made the whole
  page silently dead on `Tapeless`, and on `Clean` — the default — Drive multiplied by 4 and then
  divided by 4 around a model that does not saturate: a no-op. Tapeless and Clean now soft-clip
  under Drive; the coloured models keep their own curves. Tapeless still has no hiss floor and no
  head loss or bump: it bypasses the machine, not the knobs. (Thanks Neuro.)
- **Input channels** (`InCh`, Settings page 1: Stereo · Left · Right · Sum). A mono synth on one jack
  used to record on one side only; `Left` copies it to both. Replaces the second `InMon`, which was
  the same parameter as the Input FX menu's `Mon`. (Thanks CTribe.)
- **InSrc says when a stem is unavailable.** `S1–S4` / `M1–M4` need host 1.4 with
  `link_audio_publish`; below that they fell back to Line without a word, so the selector looked
  broken. It still falls back, and now says so on screen while Settings is open. (Thanks Hannes.)
- **Sends are stereo.** They were a mono sum, so every delay-type Palette effect (Collage, Cascade,
  Reels, Reverse) came back mono on a stereo loop; the reverbs only hid it because they generate
  their own width. Sends are now stereo and post-pan, so the bus gets what you hear from the loop.
- **Tape Drive pushes.** Up to +24 dB into the curve with makeup, instead of +12 dB and exact unity,
  which never reached the knee from a line-level source.

## v0.9.0 — 2026-09-20

The biggest release since the Overtake conversion: the Character system rebuilt from manufacturer
documentation, the Output stage made one contiguous block, the tape models re-derived from head
physics, a long list of click fixes, and a hardware-profiled CPU pass that took the engine from
56% / 74% (avg / peak, 12 loops × 4 playheads) to 41% / 43%.

### Character and the Output page
- **Thirteen Character voicings, ordered by grit** — `Off · 962 · Air · SSL · Neve · Trident ·
  Studer · API · Ampex · MPC · S950 · SP12 · Emu`. Every voicing was re-derived from manufacturer
  documentation (manuals, schematics, published specs) and measurements, with each figure tagged
  as documented, measured or chosen: [CHARACTER-RESEARCH.md](CHARACTER-RESEARCH.md). `Juno` became
  **Air** (Focusrite ISA 110) and `Console` became **Trident** (A-Range); **S950** is new.
- **Character now sets the whole output stage**, not an EQ curve: the converter (bit depth,
  sample-rate hold, µ-law companding on the MPC60), the saturator's curve and asymmetry, the
  reconstruction filter, the pole count of LoCut/HiCut (2, 3 or 6), how the loops are read
  (Hermite → linear → drop-sample for the SP-1200 / SP-12), and **Glue** and **Limit** modelled on
  the dynamics unit each machine shipped beside — Neve 2254, API 2500 (with THRUST), the SSL bus
  compressor, the A800 and ATR-102's tape compression.
- **The saturator is anti-aliased** (first-order ADAA) so the grit you hear is the machine's
  transfer curve, not fold-back; that is what keeps thirteen voicings from blurring into one.
- **Settings page 2 is now "Output"**: gSat, LoCut and HiCut moved *after* the punch bank so the
  page is one contiguous stage in signal order — Out · LoCut · HiCut · PWide · Char · gSat · Glue
  · Limit.
- Sessions store the Character by **name**; older sessions with numeric indices are remapped.

### Tape models and the record path
- **Tape HF loss derived from head physics** (gap length × speed), with a speed-scaled **head
  bump**, so Reel15 / Reel7 / Reel3 finally differ where they should: in the bass and the top.
- **VHS split in kind**: `VHS1` is the Hi-Fi track (depth-modulated FM carrier with its companding
  noise reduction, bright but pumping — the compander is now a full compress-and-expand pair at
  unity); `VHS2` is the linear edge track, slow and dark.
- **Generations cascades the selected machine's own loss filter once per pass**, so a fourth
  generation of a Reel3 is darker than one of a Reel15.
- **Studer 961/962 channel EQ** rebuilt from the service manual's fan-equaliser curves (Bass and
  Treble had been doing nothing); Mid follows the published presence spec.
- **Dither** on every loop-buffer write and on the final 16-bit output.

### Playback and interpolation
- **4-point Hermite reads** on every playhead (was linear) with a **rate-aware anti-imaging
  filter** for slowed heads — the high ringing on slowed loops is gone. Wow/flutter reads are
  Hermite too.
- **Scan is exactly 2.0×** — one octave up, not the old 1.8-octave near-miss.
- Attack/Decay were inaudible (the transport froze on pause, so the release faded DC); the tape
  keeps rolling through the release now. Un-pause restarts from Start and re-phases all four heads.
- **Self** input source: Loopex's own master output, one block old, for resampling the whole mix.

### Click fixes and level
- Reverse punch, Seed (now morphs between seeds at the next slice boundary), Smear / Haze / Strum
  (16-grain pool with voice stealing and window-power normalisation), Palette FX switching
  (outgoing fades as incoming fades in, sends and the PalFX punch), the overdub seam crossfade, and
  DC blocking on Drift, Shimmer and Self.
- **Palette level matching**: Drive, Fuzz, Howl and Sweeten used to sit many dB apart; every effect
  is now loudness-matched to the dry it replaces.
- A `bq_reset` that produced a *muting* biquad rather than a bypass — the cause of one total
  silence — is fixed at the source.

### CPU pass (profiled on the Move)
- **Per-loop pitch shifters run on their own worker thread** (`lpx-ps`, SCHED_OTHER, cores 0–2; 0.9.2
  splits it into two, `lpx-ps0` / `lpx-ps1`, after a full randomise with 13+ pitched loops overran one).
  A Signalsmith hop cost ~600 µs — a quarter of the audio callback's slack — for one pitched loop;
  it now costs the callback two `memcpy`s. Results are collected four blocks later (latency 61 ms,
  cancelled by the head-nudge as before); a missed deadline rides that loop to dry over 3 ms
  instead of clicking, and is counted.
- **The waveform display scan moved off the audio thread**. It read ~15,000 scattered samples of
  a 45 s loop on every 12th UI tick, invisible to the CPU meter but paid for by the callback.
- Per-sample work that only changed at knob rate is cached: the pan law, `pow(2, pitch)`, the
  Disintegration fade, the Master Clock's snap search and ratio (and the clock's shifter is
  bypassed at unity); the DJ filter no longer computes a 6-pole ladder just to multiply it by
  zero; idle voices are skipped at the call site; a unity-rate fast path returns the sample
  directly when the interpolation would anyway; Drift's four LFOs use a polynomial sine.
- **A per-stage profiler** is compiled in (ARM `cntvct_el0` probes, ~0.3% overhead): touch
  `/data/UserData/schwung/loopex-prof.on` on the Move and the worker writes avg/peak µs per stage
  to `loopex-prof.txt` once a second. The CPU meter (`get_param("cpu")`) gained a peak-hold.
- FTZ is now enabled on all three threads; the semitone hand-off is atomic.

### UI
- Two-line REC / LOOP / DUB counter with two-decimal times.
- Waveform view: an empty slot no longer shows the previous loop's waveform, and a loop that is
  still loading is never labelled `EMPTY LOOP`.
- Rename: Settings page 2 → **Output**; Character entries reordered by grit.

## v0.8.7 — 2026-09-19
Multi-grain freeze/wash + Grain-0 defaults, Keys/Ctrl MIDI modes with direct channel→loop
mapping, DJ-filter deferred-swap declick.

## v0.8.6
HeadMix page (per-head Vol/Pan).
