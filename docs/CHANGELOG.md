# Changelog

## v0.9.2 — 2026-09-21

- **Sessions live in the Move's UserLibrary and save loops as WAV.** Sessions moved from
  `/data/UserData/schwung/loopex-sessions` to `/data/UserData/UserLibrary/Loopex` — where the
  Schwung Manager's file browser shows them, next to Magnéto's recordings — and each loop is a plain
  `sessionNN/sNN_loop01.wav` … `sNN_loop16.wav` (16-bit stereo 44.1 kHz; two-digit numbers so
  the Manager's file browser sorts them). Existing sessions are moved on first launch (an atomic
  rename, nothing copied) and every old `tNN.raw` is converted to WAV, then removed. (Thanks Hannes.)
- **Clear** on the Sessions page (K5): wipes all sixteen loops after a confirmation, settings
  untouched, header back to *New*. K5 is also *NO* inside the popup, so a second turn cancels
  rather than wipes. (Thanks Hannes.)
- **Reset** on the Sessions page (K6): the selected loop back to factory — audio (still undoable)
  and every setting — after a confirmation. The defaults are the very same function
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
- **Per-loop pitch shifters run on their own worker thread** (`lpx-ps`, SCHED_OTHER, cores 0–2).
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
