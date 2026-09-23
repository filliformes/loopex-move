# Loopex — Claude Code context

16-track asynchronous **stereo tape looper** for Ableton Move, built as a Schwung
**Overtake** module. An instrument for ambient & experimental music; every gesture click-free.
The name is a contraction of *loop* + *experimental*.

| | |
|---|---|
| Module id | `loopex` (name `Loopex`) — **never call it "LPX"** in prose/docs |
| Type | `component_type: "overtake"`, `api_version: 2` |
| Capabilities | `audio_in`, `midi_out`, `suspend_keeps_js`, `button_passthrough: [85]` |
| Language | C (`src/loopex.c`) + QuickJS UI (`src/ui.js`) |
| Format | 44100 Hz, 128-frame blocks, stereo int16 |
| Author | **Filliformes** |
| Repo | `filliformes/loopex-move` (branch `main`); local folder `loopbox-move/` is **legacy naming** |
| Install path | `/data/UserData/schwung/modules/overtake/loopex/` |

> There is a dedicated **`/loopex` skill** with deep references (architecture, surface,
> workflow). This file is the in-repo compressed context. `design-spec.md` holds the original
> design rationale (parts of it predate the Overtake conversion); `OVERTAKE-SDK.md` is the
> reverse-engineered SDK reference.

---

## Sonic intent
A creative tape looper where recordings are living, degradable material. Every stage adds
character: tape models colour the input, the per-voice chain sculpts playback, Stability
compounds degradation, Disintegration dissolves loops into abstraction, Drift remembers and
recombines. References: 1010music BlackBox, Kinotone Ribbons, Puremagnetik LAPS, Chase Bliss
Blooper / Mood mk2 / Generation Loss mk2, Hologram Microcosm, Soma COSMOS, Studer 962,
Airwindows, norns loopers. **Not** a clean digital looper, not a sampler, not (only) granular.

---

## DSP architecture
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
Overdub modes: **Replace** / **Multiply** (default, additive with Decay) / **Disintegration**
(the loop's FX are re-applied each pass, so it dissolves).

### Key constants (`src/loopex.c`)
```
SR 44100   NUM_VOICES 16   LOOP_SECONDS 45   POLY_VOICES 8   NUM_PREAMP 13
PUNCH_BUF 88200 (2 s)      NUM_PUNCH 16      NUM_PSLOTS 5    NUM_CHOP_PAT 32
FXSEQ_STEPS 16             NUM_SLOTS 64      MF_NVOICE 12    DRIFT_N 4
```

### Subsystems
- **Playheads** — 4 per voice (Off/Fwd/Bwd/Ping/**Jump**), `1/sqrt(n)` sum, 64-sample declick
  crossfade on any jump. Head 0 is the main head (Scatter/Seed/scrub drive it).
- **DJ filter** — LP left (analog voicing via `mf_run`, 12 models) / HP right (biquad). Both
  crossfade to dry across ±0.08 of centre and **the engine swap is deferred until the wet blend
  reaches ~0** (`djWet`/`djWetTgt`) → pop-free at any sweep speed.
- **Palette sends** — two buses, `Off` + 24 effects + 4 reverbs (Plate/Quartz/Prism/Veil) =
  `PFX_COUNT` 29; block-processed, 1-block latency; effect swaps go through the worker and
  **morph** (outgoing fades as incoming fades in); every effect is level-matched to its dry.
- **Punch-in FX** — 16 pads, **per-sample, zero-latency, up to 5 in series** over a 2 s ring.
  Stretch/Freeze is a 4-grain wander-wash (randomised lengths + backward wander + per-grain
  stereo, weighted-average sum).
- **Drift** — 4 coprime delay lines with drifting taps and a Hadamard cross-mix; feedback capped
  below unity, silence bleed after ~8 s.
- **Sessions** — 64 slots in `/data/UserData/UserLibrary/Loopex/`; **all disk I/O on a
  `SCHED_OTHER` worker (`lpx-sio`) pinned to cores 0-2**, joined in `destroy_instance`. Audio is
  `sessionNN/sNN_loopNN.wav` (both 1-based, as the pads and slots are numbered) (16-bit stereo WAV, interleaved on the worker in 4096-frame chunks; `wav_write`
  / `wav_read`); pre-0.9.5 `tNN.raw` (L block then R block) is read as a fallback and removed once
  a `.wav` has been written over it. Sessions page K5 = **Clear** (`clearAll`: `voice_clear` + `voice_defaults` on all 16) and K6 = **Reset** (`resetSel`) / K7 = **Reset All** (`resetAll`, one undo group) (`resetSel`:
  `voice_defaults` with state/loopLen/playPhase/muted preserved - settings only, audio stays; the same
  function `create_instance` uses), both with the Del-style
  confirm popup (K5 = NO / K8 = YES). The
  waveform display scan (`wave_compute`) and the Disintegration pass also run there.
- **Pitch shifters** — Signalsmith Stretch **2048/512** (1024/256 was rejected by ear) on two
  workers **`lpx-ps0` / `lpx-ps1`** (even / odd voices; SCHED_OTHER at nice -10 - the process has CAP_SYS_NICE - cores 0-2, one `sem_post` each per
  block). One worker overran with 13+ shifters (`late` climbing ~25/s); each hop is ~600 µs. At nice 0 both workers
  were stalled >23 ms every ~2 s (12 pitched loops); nice -10 took that to ~0. Per-voice 24-block queue (PS_NQ), PS_D 8
  indexed by the global block number; the callback submits a block and collects the result
  `PS_D=4` blocks later (`psLat` includes it, the head-nudge compensates). Missing result ⇒
  keep the last block and ride `psReady` to dry over ~3 ms; `late=` counted in the profile.
  Each hop costs ~600 µs — on the callback that was a quarter of the slack per pitched loop.
- **Character** — `MEQ_DEF[13][23]`: `Off · 962 · Air · SSL · Neve · Trident · Studer · API ·
  Ampex · MPC · S950 · SP12 · Emu` (grit order). Each row sets EQ, saturator (ADAA, clip/asym),
  converter (bits / crushHz / µ-law compand), reconstruction filter (bwHz), LoCut/HiCut pole
  count, Glue (atk/rel/knee/dist), Limit ceiling, THRUST and the read-interpolation grit. Sessions
  store the **name**; legacy numeric indices are remapped. Sources: `docs/CHARACTER-RESEARCH.md`.
- **Profiler** — `cntvct_el0` probes per stage inside the sample loop; the worker dumps avg/peak
  µs to `/data/UserData/schwung/loopex-prof.txt` once a second while `loopex-prof.on` exists.
  The Overtake CPU meter (`get_param("cpu")` = "avg/peak") times **`render_block` only**, and
  its denominator is the full 2902 µs block, so ~82% is the real danger line.
- **InSrc (Link Audio)** — `Line · Master · S1-4 · M1-4`. `S1-4` read Schwung's published stems
  from `/schwung-pub-audio` ("BPAL"); `M1-4` read the reconstructed Move tracks from
  `/schwung-link-in` ("LAIN"). Both read-only, with a **private cursor** (never touch `read_pos`).
  A stem that is not there (host < 1.4 or `link_audio_publish` off) falls back to Line; `inSrcLive`
  (get_param) is 0 then and the UI says so while Settings p1 is open. `inChan` (Stereo/Left/Right/Sum)
  is applied right after the source read.
- **Dynamic sampler** (0.9.5, `dyn_*` in loopex.c): per sample on the finished input, before the
  record loop. A capture = `voice_clear` (undoable) + `VS_RECORDING` + 100 ms pre-roll from a ring,
  closed to `VS_PLAYING` at Size (tempo via `punch_beat()`) or after 200 ms of quiet (Free/Phrase);
  then `voice_randomize(v, dynError)` and a Sustain timer (`dynDie`, paused when it expires). Target
  = selected pad + rotating cursor within Spread, skipping muted/armed/recording pads; a pad with
  audio is skipped unless `dynOverwrite` - with none free the sampler parks (`dynWait`) and raises
  `dynFull`, which the UI turns into the CONTINUE DYNAMIC LOOPING / AND OVERWRITE PADS? popup (K6 = overwrite on, K5/Back = mode
  Off; it takes over any view and holds the view timeout while up). Changing the mode resets the permission. Modes 4 Pitch / 5 Novelty exist in the enum but
  are hidden in ui.js until their worker-side analyser exists. Clock counts Move's MIDI clock
  (0xF8/0xFA in on_midi, before the len<3 guard; `clkFire` on each Size boundary, Free = 1 bar;
  free-running fallback when no tick for 0.5 s) and keeps counting while a capture records; a
  boundary closes the capture and opens the next. Sustain expiry sets `dynSpent` (pad free, no ask);
  voice_tap/odub clear it. Only the latest capture's undo records survive (`UndoRec.dyn`). Phrase =
  400 ms gap, 1 s minimum. No triggers while InSrc = Self. Capture long-press (600 ms) toggles
  Off <-> last mode; the white-only LED blinks while listening.
- **ASYNC / SYNC** (0.9.5, Sessions K8 `syncMode`): a beat grid `beatPos` ALWAYS runs (tempo per
  block from get_bpm; pulled onto Move's 0xF8 clock once 0xFA Start / 0xF2 Song Position has locked
  it), so switching never starts anything. `sync_events` fires on beat/bar crossings: syncPend 1 =
  record on the next beat (a tap <=1/8 beat late starts now with the pre-roll ring, which now always
  runs), 2 = play from Start on the next bar; Start restarts playing loops. Tap-close rounds to bars
  (`sync_round_len`, `recTarget`). Varispeed `tempoSm` = bpm/recBpm glided (recBpm saved as v<N>.bpm).
  Jump/Scatter on 1/16s (`loop_beatS` = the loop's own grid); Stumble/Drift note values; FX-seq steps
  from the grid; Pump phase slews onto it; punch slice mechs wait live (`gridWait`, ring still
  recording) to the next grid line, then cut the slice that just played (`punch_slice_arm`); Rate/Len snap
  to note values (`nv_snap`, Reverse via `rev_el` so its hop is the note); pressure steps (`press_step`,
  hysteresis) and only re-latches on a full slice line (`runT`/`fullT`); Mosaic/Strum fire on grid cells
  (`gCell`); punch LFO in note values, phase-slewed to the bar. ASYNC pressure: asymmetric smoothing
  (in ~15 ms, out ~65 ms), Chop/Mosaic continuous, Strum humanised + probabilistic octave. Dynamic keeps a
  separate SYNC Size (`dynSizeS`, default 1 bar); Free captures are rounded + enter on the bar;
  Sustain in bars. Switching to ASYNC flushes pending starts and closes a take waiting for its bar.
  UI: values re-quantise only when turned; Speed/head speeds on ratios, Start 1/16, End note lengths
  (`v_grid`), header bars.beats (`v_bars`), beat lines on the waveform, pads blink while pending, and
  taps poll instead of predicting the next state.
- **Tape Wear** (0.9.5, `wear_apply`): per voice a STEREO damage map `wmap[2][WEAR_MAX]` of ~5.8 ms cells
  (weakness 70% shared / 30% per track, edge track L ×1.15, 10% cross-spill; file = nc, L, R)
  (WEAR_CELL 256), non-destructive. A head crossing a cell wears it: d += rate·(seed+4d)·(1−d),
  seed from a per-cell hash (weak spots), 15% to the neighbours; rate = 0.0005·400^knob per
  crossing (0 = frozen). Read: interpolated d → Wallace spacing-loss one-pole (fc = 5220/(6d) Hz),
  level (1−d)(1+d/2), per-lap per-track flicker, rare crackle. Zeroed at a new take and by
  voice_defaults (Reset heals); copied by clone; saved as `sNN_wearNN.bin`; carried by undo records
  (`wmap`/`wmapOn`). Knob `v_wear` / `v<N>.wr`, Tone page K8 (replaced the Heads shortcut).
- **Loop window** (0.9.5): End is a LENGTH, `effLen = loopEnd*LEN` clamped to `LEN-effStart` (was
  `loopEnd*avail`). Every jump crossfade is `XF_N` = 256 samples (was 64); Seed slices >= XF_N;
  Ping heads reflect by the overshoot. Revert point before this: commit 906f740.
- **Undo history** (0.9.5): `UndoRec hist[32]` in `loopex_t`, each with its own stereo buffer + gen
  stamps (calloc'd: address space until a loop records into it). Clear all = one AUDIO record per
  pad carrying the settings too (`hasSet`, `undo_audio_set`). Trigger knobs fire once per 400 ms. `undo_audio` SWAPS the pad's buffer pointers with
  the record's (clear / arm / Dynamic overwrite / Clear all - via `loop_clear`); `undo_settings`
  snapshots a `VoiceSettings` (randomise, reset via `loop_reset`); an overdub start pushes an
  UR_OVERDUB record whose diff the record loop fills (`odRec`). `undo_pop` restores every record of
  the newest group on the callback, except overdubs, which go PENDING to the worker (`undoReq` =
  slot+1). `histGroup++` per gesture. Voice and record buffers are one permuted set: destroy frees
  both. `undoAvail` = "n:Tx what", `undoMsg` after a pop. Undo+pad no longer overdubs; Shift+Undo+pad
  = `reset:N`.
- **Randomiser** (`voice_randomize`, also Rnd Pad / Rnd All): 75% speed-only transposition (no shifter
  engaged - a full randomise that pitched 11 loops overran the shifter worker), else Speed and Pitch as complementary
  musical intervals (T = heard, S = tempo, both from {0,±3,±4,±7,±12,±15,±16,±19,±24}, P = T-S within
  ±12, every valid pair enumerated and picked uniformly), unison 30% of the time; head speeds untouched; heads 2-4 on/off with pan only when
  active; Volume untouched; send/sendB/comp/sat <= 0.6; Start/End window >= 50% (or left whole).
- **Input Tape knobs act on every model** (0.9.1). They were gated on `preModel>0`, which made the
  page dead on Tapeless and Drive a x4-then-/4 no-op on Clean. Tapeless/Clean soft-clip under Drive.

---

## Surface
- **Pads are notes 68–99** (`padNoteFor`: `68 + row*8 + col`, 4×8). Left 16 = loops,
  right 16 = punch FX. *(Older notes in this file claimed 36–51 — that was wrong.)*
- **5 loop pages** (Up/Down): `Loop · Texture · Tone · Heads · HeadMix`.
  HeadMix = per-head Vol/Pan; **H1V/H1P are the same params as page 1's Vol/Pan** by design.
- **8 menus** — track buttons 1-4 → Input (p1 Tape / p2 EQ) / Perform / Send FX / Settings; Capture →
  **Dynamic** (0.9.5; Input Tape merged into Track 1); ≡ → Sessions; ● → Drift; ✕ → FX Seq. **Perform and Settings are two-page.**
- **Settings p1** ArmTh · ODub · LpFlt · Root · **InCh** (0.9.1; was a duplicate InMon) · InSrc · MIDI · MidiO —
  **p2 is named "Output"** (`MENU_PAGE_NAMES`): Out · LoCut · HiCut · PWide · Char · gSat · Glue · Limit,
  in signal order after the punch bank. **Perform p2:** Cut · Reso · FChar · Clock · ClkMd · ClkAt ·
  Pump · PmpRt. Dropout (`dropAmt`) exists in the DSP but is **not on any page**.
- Gestures: tap = Empty→Rec→Play⇄Pause; double-tap = Overdub; hold = Clear (Undo restores);
  Shift+tap = speed ½/1/2×; Mute+tap = quick-mute; Copy+pad+pad = clone; Loop+pad = loop length;
  Shift+punch pad = latch; Undo+punch pad = reset that effect.

### ⚠ `ui.js` traps
- **`MENU_DEFS[1]` and `[2]` are swapped at runtime** right after the literal
  (`{ const t = MENU_DEFS[1]; MENU_DEFS[1] = MENU_DEFS[2]; MENU_DEFS[2] = t; }`). Literal order
  is Input / Sends / Perform / Settings; **runtime** order (which `MENU_NAMES` and
  `MENU_PAGED = {0:true, 1:true, 3:true}` assume) is Input / **Perform** / **Sends** / Settings.
- **Punch defaults live in `ui.js` `punchVals`**, not the DSP — the UI pushes them on every pad
  press, so the DSP's `punchParams` init is only a fallback. `PUNCH_DEFAULTS` (Undo+pad) is a
  snapshot of `punchVals`. Stretch/Freeze default **Grain = 0**.

---

## Parameters (string bridge)
JS drives the DSP entirely through `set_param(key,val)` / `get_param(key)`.
- **Per-voice (selected track):** `v_start v_end v_reverse v_pitch v_filter v_pan v_volume
  v_decay v_sat v_wowflut v_sendA v_sendB v_scatter v_glitch v_tilt v_eqBass v_eqPresFrq
  v_eqPresAmt v_eqTreble v_djReso v_atk v_rel v_comp v_clock`, `v_ph<1-4>mode` / `v_ph<1-4>spd`,
  `v_hvol2..4` / `v_hpan2..4`. Read-only: `v_state`, `v_loopLen`.
  Indexed form for state restore: `v<0-15>.<sub>`.
- **Globals:** master/output (`masterVol masterLoCut masterHiCut masterEQ masterGlue tapeLimit
  globalSat masterComp`), behaviour (`overdubMode armThresh rootNote stability globalWowFlut
  loopFiltMode selTrack`), I/O (`inSource inputMonitor inputGain inLow inMid inMidFreq inHigh
  inHighFreq inLowFreq inChan`), tape (`tape*`), dynamic (`dynMode dynSense dynSize dynSpread dynError
  dynSustain dynOverwrite dynResume`; read-only `dynCount dynFull`; commands `rndSel rndAll`), perform (`st* mf* mClock* perfTrem* dropAmt`), drift (`drift*`),
  punch (`punchWidth pfx pflfo punch punchPress`), sequencer (`fxseq*`), MIDI (`midiIn midiOut`).
- **Commands:** `cmd` (`tap/odub/clear/unclr/undo/mute/sel/arm/clone`), `session`, `scrub`,
  `headpos`, `jump`, `scan`, `tapeHold`, `clearSel`, `clearAll`.
- **Read-only:** `states mutes wave heads cpu inputPeak undoAvail sessNames sessStatus sessName
  state chain_params ui_hierarchy`.
- **State blob:** `get_param("state")` emits newline `key=value` lines (globals + compact
  `v<N>.*`); `set_param("state")` replays each line through the same dispatcher. It returns
  **−1 rather than truncating** on overflow. Enums `match_enum` the name **before** `atof`.

---

## MIDI
**`Settings → MIDI` is a 3-way mode: `Off` (default) / `Keys` / `Ctrl`.**
- **Keys** — keyboard poly. `on_midi` maps **channel n → loop n** (`msg[0] & 0x0F`), 8 voices.
- **Ctrl** — LaunchControl XL, handled in `ui.js onMidiMessageExternal` (gated on the cached
  `midiMode`). Track Focus notes **68-75 / 76-83 = tap**; Track Control notes **36-43 / 44-51 =
  mute**; CC **1-32** (loops 1-8) and **41-72** (loops 9-16) in rows of 8 → Speed / Filter / Pan /
  Volume. **`MidiO`** mirrors transport to the LCXL LEDs on **MIDI channel 2** (channel 1 would
  collide with Move's own pad notes). Templates: `BlackBox 1-8.syx`, `BlackBox 9-16.syx` — set
  both to channel 2.
- **Known limitation — 4 loops, not 16.** A Schwung Overtake module only receives external MIDI
  through the host's **four forwarding slots' "out"**, so only 4 channels arrive. Confirmed by
  Charles Vestal: *"modules only listen to the out of schwung modules, that's the only mailbox we
  have."* `/schwung-ext-midi-remap` exists (16-entry passthrough) but is **disabled and unwired**,
  and `MOVE_MIDI_SOURCE_FX_BROADCAST` is audio-FX-only. **The mapping already handles all 16** if
  a host ever delivers them — do not "fix" this in the module.

---

## Critical implementation notes
- **Every entry point runs on the SPI audio callback** (`create_instance`, `destroy_instance`,
  `set_param`, `get_param`, `on_midi`, `render_block`). No allocation, file I/O, logging or locks
  on the recurring paths. One-time `calloc`/`mmap` in create; `free`/`munmap`/`pthread_join` in
  destroy. No mutexes anywhere — lock-free atomics only; the one syscall the callback makes is
  `sem_post` (a non-blocking futex wake) to the shifter worker.
- **`suspend_keeps_js`: a plain Back only SUSPENDS.** Old JS *and* DSP stay live, so a new build
  silently does nothing. **Full exit = `Shift + Volume + Jog-click`.**
- **Denormals:** FPCR FZ is **per-thread** on aarch64 and `-ffast-math`'s `crtfastmath` does not
  set it on the callback thread → `lb_enable_ftz()` runs every block **and at the start of both
  workers** (the phase vocoder's bins decay to denormals on a silent loop). The build also uses
  **`-fno-finite-math-only`** so the `x!=x` NaN guards in `lb_clampf/lb_clampd` survive — do not
  remove it.
- `frames` is clamped to ≤128 at the top of `render_block` **before** any `frames*2` use.
- **`get_param` MUST return −1 for unknown keys** (not 0).
- Publish `loopLen` **last** with `__ATOMIC_RELEASE` when filling a buffer, so render never sees
  a half-written loop.
- Feedback paths capped below unity + saturated in-loop; filter cutoffs clamped inside
  (0, Nyquist).

## Move hardware constraints (never violate)
- 128 frames @ 44100 Hz (~2.9 ms); the module's slice of the callback is ~2370 µs.
- Audio is int16 stereo interleaved.
- No heap allocation / logging in the render path. No FTZ by default on ARM.
- Files on the device must be owned by `ableton:users`.
- **Memory:** ≈127 MB for the 16 stereo loop buffers (16 × 45 s × 2 ch × int16), plus ≈12 MB of
  shared overdub-undo buffers, ≈3.5 MB of punch rings, ≈1.7 MB of Drift lines, ≈0.5 MB of
  shifter queues — all allocated once in `create_instance`.

---

## Repo map
```
overtake-shell/
  module.json              Overtake manifest (id / capabilities / version)
  src/
    loopex.c               the engine: voices, playheads, punch FX, Stumble, Drift, sessions, master
    palette_fx.c/.h        Palette send engine (Off + 24 effects + 4 reverbs)
    fx_clouds.cc           Clouds-based Space / Bloom (C++)
    pitch_shift.cc         Signalsmith Stretch wrapper (per-loop Pitch)
    warps_data.c           Warps wavetables (Fold / Shift)
    shell_dsp.c            minimal smoke-test shell (not in the shipped link line)
    ui.js                  QuickJS Overtake UI (pads, knobs, screens, LEDs, gestures)
  include/plugin_api_v1.h  host API (v1 + v2)
  vendor/                  clouds_engine, signalsmith, signalsmith-stretch
  scripts/                 build.sh, install.sh, Dockerfile
README.md                  maintained feature reference + Known limitations + Credits
docs/MANUAL.md             long-form manual
docs/index.html            GitHub Pages site (filliformes.github.io/loopex-move)
docs/CHARACTER-RESEARCH.md every figure behind the 13 Character voicings (DOC / MEASURED / CHOICE tags)
docs/CHANGELOG.md          release history
design-spec.md             original design rationale (partly pre-Overtake)
OVERTAKE-SDK.md            reverse-engineered Overtake SDK reference
BlackBox 1-8.syx / 9-16    LaunchControl XL templates (set to MIDI channel 2)
release.json               version + release asset URL for the catalog
```

## Build & deploy
```bash
cd overtake-shell
bash scripts/build.sh      # Docker cross-compiles dsp.so (aarch64) + validates ui.js
bash scripts/install.sh    # scp + atomic-rename to move.local
```
Then on the Move: **full-exit (`Shift + Volume + Jog-click`) and reopen** — a plain Back only
suspends. `MOVE_HOST=ableton@172.16.254.1` overrides the target. The installer stages to
`/data/UserData` and `mv -f`s into place; **never write straight over a mapped `.so`**.

## Release
1. Bump the version in **both** `overtake-shell/module.json` **and** `release.json`.
2. Commit, push `main`, then `git tag vX.Y.Z && git push origin vX.Y.Z`.
3. `.github/workflows/release.yml` verifies tag == `module.json` version, builds, and publishes
   `loopex-module.tar.gz`.
4. Catalog: PR to `charlesvestal/schwung` → `module-catalog.json`. The fork is
   **`filliformes/move-everything`** and its `main` has diverged, so **branch off `upstream/main`**.
   `tags` must come from `taxonomy.json` **and be sorted**; `min_host_version: "1.3.0"`.
5. Run **`/dsp-review`** over `src/loopex.c` before tagging.

## Definition of done — keep all four in sync
Any user-visible change must land in **all** of these before it's finished:
1. **`README.md`** — feature sections, the loop-page table, Controls tables, Signal chain,
   Known limitations, Credits.
2. **`docs/MANUAL.md`** — menu tables and the matching prose section.
3. **`docs/index.html`** — the site (nav list, hero pills, `#pages` table, `#credits`); reuse the
   existing CSS classes, no new dependencies.
4. **`CLAUDE.md`** (this file) — if the architecture, params, surface or constraints moved.

Counts must agree across all four (e.g. **24 Palette effects + 4 reverbs**, **32** Chop patterns,
**64** session slots, **5** loop pages, **13** tape models, **13** Character voicings). Push the
docs with the code, and add the release to `docs/CHANGELOG.md`.
