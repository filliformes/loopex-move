# Changelog

## v0.9.2 — unreleased

The biggest change to how Loopex is *played* since the Overtake conversion. It adds a **SYNC** mode
that locks everything to Move's bars (Loopex stays asynchronous by default), a **Dynamic sampler**
that turns your input into loops by itself, **Tape Wear** (your loops disintegrate as they play) and
**musical randomisers**. Undo goes back **32 steps**, sessions are saved as **WAV** where the Schwung
Manager can see them, and a number of gestures moved. Read *Upgrading from 0.9.1* and *Where things
moved* first: some habits from 0.9.1 now do something else.

### Upgrading from 0.9.1 — read this first
- **Your sessions move and become WAV files.** On first launch every saved session moves from
  `/data/UserData/schwung/loopex-sessions/slotN` to `/data/UserData/UserLibrary/Loopex/sessionNN`, and
  each loop becomes `sNN_loop01.wav` … `sNN_loop16.wav` (16-bit stereo, 44.1 kHz). Nothing is copied:
  folders are renamed, and each old `.raw` file is deleted only after its WAV has been written. **This
  is one-way.** 0.9.1 can't read the new sessions, so don't downgrade without a backup. If a
  `sessionNN` folder already exists in the new place, the old slot with that number stays where it was
  and is no longer listed.
- **Old sets and sessions sound the same.** A 0.9.1 state is translated on load:
  - The Input LoCut and the high-shelf corner used to sweep linearly and now sweep in log. Their
    values are converted, so the frequencies don't change.
  - The low shelf used to be fixed at 120 Hz and now has its own knob (LowF). It opens at 120 Hz.
  - End is now a length (see *Loops*). Every loop's End is converted, so its window keeps its length.
- **Overdub is no longer Undo + pad.** Hold the loop's **step button** instead (about 0.6 s). Undo +
  pad now does nothing and reminds you where overdub went.
- **Capture no longer opens Input Tape.** It opens the new **Dynamic** menu. Input Tape and the Input
  EQ now share **track button 1**, as two pages.
- **The Tone page's K8 is now Wear.** It used to be the *Heads ▸* shortcut. The Playheads page is still
  one press of **Down** away, or a second tap on the loop's step button.
- **Loopex is still asynchronous by default.** Nothing snaps to a grid unless you switch Sessions → K8
  **Mode** to **SYNC**.

### Where things moved

| Control | 0.9.1 | 0.9.2 |
|---|---|---|
| Track button 1 | Input FX (one page) | **Input**: 1st press *Input Tape*, 2nd press *Input EQ*, 3rd press closes; Up/Down also flip its pages |
| Capture, short press | Input Tape menu (on press) | **Dynamic** menu (opens on release) |
| Capture, hold ≥ 0.6 s | — | **Dynamic sampler on/off** (turns on the last mode used; *Level* the first time) |
| Loop step button, hold ≥ 0.6 s | — | **Overdub** on/off |
| Undo + loop pad | Overdub | nothing (shows `overdub: hold step N`) |
| Shift + Undo + loop pad | — | **Reset** that loop's settings (audio stays), no confirmation |
| Tone page, K8 | Heads ▸ | **Wear** |
| Sessions K5 / K6 / K7 / K8 | — | **Clear** / **Reset** / **RstAll** / **Mode** (ASYNC · SYNC) |
| Dynamic K7 / K8 | — | **RndPad** / **RndAll** |

**Input, page 1 — Input Tape:** Mon · Tape · Trim · Drive · Wow · Flut · Hiss · Gen
**Input, page 2 — Input EQ:** LoCut · HF · LowF · Low · Mid · MidF · High · HiF
**Dynamic (Capture):** Dyn · Sense · Size · Spread · Error · Sustn · RndPad · RndAll
**Sessions (≡):** Slot · Save · Load · Del · Clear · Reset · RstAll · Mode

(In 0.9.1, Input FX was Mon · Style · Trim · Low · Mid · MidF · High · HiF, and Capture's Input Tape
was Tape · Drive · Wow · Flut · HF · LoCut · Hiss · Gen. The tape model knob is now called **Tape** and
exists once.)

### New: ASYNC / SYNC (Sessions page, K8 Mode)
Loopex stays an **asynchronous** looper by default: loops are exactly as long as you play them and
nothing waits for a beat. Set **Mode** to **SYNC** and it follows Move's bars and beats (4/4 bars,
triplet values included).

**Recording and playing in SYNC**
- **A tap starts recording on the next beat.** A tap up to 1/8 beat *late* still catches the beat it
  just missed, because the start is taken from the last 100 ms of input. While a pad waits for its beat
  it **blinks red**; tap it again to cancel. A threshold-armed pad also starts on the beat after the
  input crosses.
- **A tap to close rounds the take.** A take longer than ¾ bar rounds to the nearest whole bar; a
  shorter one rounds to whole beats (at least one). Rounded down, it plays from its start at once.
  Rounded up, it keeps recording to the bar line; a second tap while it waits closes it right there,
  unrounded.
- **Un-pausing waits for the next bar.** The pad **blinks green** until it comes back in, from Start.
- **Move's Play restarts every playing loop on bar 1.** Paused loops stay paused. Until Move sends Play
  (or a song position) Loopex keeps Move's tempo on its own grid, so its bar 1 may not be Move's bar 1.
- **Loops follow tempo changes, tape-style.** Every take made in 0.9.2, in ASYNC too, remembers the
  tempo it was recorded at. In SYNC, speed and pitch bend together by *Move's tempo ÷ recorded tempo*
  (gliding over about 0.1 s, limited to ¼×–4×). So switching to SYNC after changing Move's tempo
  audibly re-pitches those loops; it never jumps. Loops recorded in 0.9.1 have no stored tempo and
  keep their speed.
- **Not quantised:** starting or stopping an overdub, pausing, and a take that runs into the 45 s limit.

**Knobs in SYNC**
- **Speed and the playhead speeds** move in tempo ratios: ¼× ½× ⅔× 1× 3/2× 2× 4×, shown as `1/4x … 4x`
  (Pitch stays in semitones).
- **Start** moves in sixteenths (shown as `37/16` counted from the loop start); faster turns skip more.
- **End** moves in note lengths, 1/16 … 8 bars (triplets included), plus `all` when the loop isn't a
  whole number of bars.
- Values only snap the next time you turn them. A value set in ASYNC shows as a number (e.g. `1.37x`)
  until then.

**What else goes on the grid**
- **Jump heads** leap after 1–8 sixteenths (ASYNC: 60–560 ms) and land on sixteenths; **Scatter**
  slices are sixteenths.
- **Perform → Step** (Stumble) runs in note values, 1/32 … 1 bar; **Drift → Rate** in 8 bars … 1/16 per
  cycle (its speed follows the tempo, its phase is free).
- **The FX sequencer** steps from the grid; **the Pump** slides its phase onto the grid.
- **Punch-in FX:** see *Punch-in FX* below.
- **Dynamic** gets its own Size (default 1 bar) and Sustain in bars: see *Dynamic sampling*.

**On screen:** the header counts `LOOP 2.1 BAR` / `REC 1.3 BAR` (bars.beats) instead of seconds; the
waveform shows faint dots on each beat and denser ones on each bar. Randomised Speeds at thirds and
fifths are real musical intervals, not tempo ratios, so they drift against the bar; octaves stay on it.

**Switching modes:** nothing that is playing jumps. The Pump and tempo following glide, values snap
only when turned, and switching back to ASYNC starts at once anything that was waiting for a beat or a
bar, and closes a take waiting for its bar line.

### New: Dynamic sampling (Capture button)
The input plays the sampler: every phrase you play lands on a pad by itself, as an ordinary recording
with the pad's tape stage, loop pages, sends and lights. Hold **Capture** (0.6 s) to switch it on or
off; the Capture light **blinks** while it listens. A short press opens its menu.

**The knobs**

| Knob | What it does |
|---|---|
| **Dyn** | Off · **Level** · **Onset** · **Phrase** · **Clock** — what starts and ends a capture |
| **Sense** | the threshold, −60 dBFS (knob at 0) to −6 dBFS (at 1); middle = −33 dBFS |
| **Size** | 1/16, 1/8, 1/4, 1/2, 1, 2, 4, 8 bars, or **Free** (until the input goes quiet). Default Free. SYNC keeps its own Size, default 1 bar |
| **Spread** | how many pads it rotates through, 1–16 (default 16), starting at the pad selected when you switched it on |
| **Error** | 0–1: how far each capture's settings are randomised (see *Rnd Pad*). Default 0 |
| **Sustn** | how long a capture plays before it pauses itself. ASYNC 1–60 s (log), SYNC 1–64 bars; the top of the knob = never (the default) |
| **RndPad / RndAll** | the randomisers (below) |

Sense reads in dB, Error in %, Sustn in seconds (bars in SYNC) or `never`.

**The modes**
- **Level:** starts when the input rises above Sense, and re-arms once it has fallen 6 dB below it.
- **Onset:** starts on a *jump* in level (a hit), so Sense works as a contrast, not a level. The next
  hit (at least 100 ms later) ends the capture and starts a new one. With Size = Free it also ends
  after 200 ms more than 20 dB under its own peak.
- **Phrase:** captures sentences. Gaps under 400 ms don't end it, it lasts at least a second, and it
  always ignores Size.
- **Clock:** captures back to back on Move's MIDI clock: on the beat, and on the bar once Move has pressed
  Play. Size = Free means one bar. A capture only starts if something is playing (within 20 dB of the
  threshold). With no clock for half a second it keeps Move's tempo by itself; in SYNC it uses
  Loopex's bar grid.
- Level, and Onset with Size = Free, end after 200 ms of near-silence (about 10 dB under Sense).
- Every capture except Clock keeps the **100 ms before** the trigger, so the attack isn't lost. A set
  Size includes that pre-roll.
- In SYNC a Free capture is rounded to whole bars and comes in on the next bar. A set Size already
  fits the grid and plays at once.

**Pads**
- A capture **replaces** what was on the pad, never stacks.
- **Muted, armed, recording and overdubbing pads are skipped.**
- A pad that already had audio is never overwritten without asking. When the range is full, the
  question `CONTINUE DYNAMIC LOOPING / AND OVERWRITE PADS?` takes over the screen (it opens the Dynamic
  menu if needed). **K6 or K8 = YES:** it carries on through the pads, overwriting them, until you switch Dynamic off.
  **K5, Back, or opening another menu = NO:** Dynamic switches off. This question is answered by the
  first knob turn after it appears.
- **Sustain frees pads.** A capture whose Sustain ends pauses (with its Release fade), and its pad
  counts as free again. If the sampler was waiting on a full range, it resumes by itself (`pad freed -
  Dynamic resumes`).
- **Tapping a captured pad makes it yours:** it is no longer Sustained or free to overwrite.
- With **InSrc = Self**, the sampler pauses instead of recording its own playback, and says so every
  couple of seconds.
- **Only the latest capture is undoable,** so a busy sampler doesn't flush your own undo steps.
  (Captures with Error > 0 add their settings as one more step.)
- **Dynamic is saved** with the set and with sessions: its mode and every knob. Reopening Loopex puts
  it back as you left it (if it was listening, it listens again; it still asks before overwriting).
  **Loading a session never switches it on:** the session's Dynamic knobs come back, the sampler stays
  off until you hold Capture.

### New: Rnd Pad / Rnd All (Dynamic K7 / K8)
One turn randomises the selected pad (**RndPad**) or all sixteen (**RndAll**), by musical rules, with
no confirmation. One Undo brings it all back.
- **Pitch, only in musical intervals.**
  - 30 % of the time Speed and Pitch stay as they were.
  - Otherwise, most of the time (75 %) only **Speed** moves, by ±3, 4, 7, 12, 15, 16, 19 or 24
    semitones: thirds, fifths and octaves, up to two octaves.
  - The rest of the time Speed and Pitch move together as a pair from those intervals, with the Pitch
    knob never more than one octave from centre.
- **Playheads:**
  - Head 1 and every head speed and volume are untouched.
  - Heads 2–4 each come on 35 % of the time, in a random mode (Fwd, Bwd, Ping or Jump); otherwise
    they are switched off. Pan is only randomised on heads that are playing.
- **The loop's FX:**
  - Each of Sat, Comp, Wow/Flutter, Scatter, Seed, Tilt, Bass, Presence and Treble is engaged half
    the time, otherwise off.
  - The DJ filter moves 40 % of the time, otherwise it goes back to the centre.
  - Sends A and B and the filter Resonance are always set, between 0 and 60 %. Sat and Comp also stay
    at or under 60 %.
- **Always randomised:** Pan, Attack, Release (up to 5 s), the Presence frequency and Reverse (50 %).
- **Never touched:** Volume, Decay and Wear.
- **The window:** half the time Start lands in the first half of the loop and End keeps at least half
  of it; otherwise the loop plays whole.

### New: Tape Wear — the Disintegration Loops, per loop (Tone page, K8)
- **Your loops can wear out** the way William Basinski's ageing tape loops did in *The Disintegration
  Loops*, where the oxide flaked off a little more on every pass over the head.
  - Every time a playhead crosses a spot on the loop, that spot loses some oxide. The treble goes
    first (a head lifted off the tape by loose flakes), then the level, until the spot drops out
    entirely.
  - Damage starts at a few weak places, grows where it started and speeds up near the end. So the
    music breaks into the *same* holes every lap and stays recognisable while it dies.
  - It is stereo, like the two tracks side by side on real tape. Each side has its own damage, mostly
    shared, partly its own, and the edge track wears a little faster. A hole often opens on one side
    first, and the stereo image wanders as the loop dies.
- **The knob is a speed, not an amount.**
  - At the top, an 8-second loop collapses in about a minute; in the middle, about twenty; near the
    bottom it takes hours.
  - Wear counts laps: shorter loops, faster speeds, more playheads and scrubbing with the jog all wear
    it faster.
  - At 0 the tape stops wearing, and the damage stays exactly where it is.
- **Shift + Undo + pad on a worn pad gives you clean tape again.** It resets that loop's settings and
  heals all its wear at once, keeping the recording. Sessions → Reset (and RstAll, and Clear) do the
  same. Turning Wear down to 0 only stops the wear; it never heals it. Changed your mind? Undo brings
  the damage back.
- **Nothing is destroyed.**
  - The recording and the saved WAV stay clean: the damage is a map laid over them. It is saved with
    the session in extra files (`sNN_wearNN.bin`), so a half-dead loop reopens half-dead.
  - It is copied when you clone a loop and undone with everything else.
  - A new recording on a pad starts on fresh tape.
  - Notes played from a keyboard (MIDI Keys mode) read the loop clean.
- **Not the Disintegration overdub mode.** That mode (Settings → ODub) still bakes the loop's own
  effects into the audio once per overdub pass. Wear works on any loop while it simply plays, and never
  touches the audio.

### Loops
- **Overdub = hold the loop's step button** (about 0.6 s). Hold again to stop. It is one-handed and
  instant. Holding a recording pad's step closes the take; on an empty loop it just says `T3 empty`. A
  short tap still selects, and a tap on the already-selected loop flips to its next page on release.
  (Thanks Hannes.)
- **End is a length now.** It used to be a fraction of what was left after Start, so sliding Start to
  the right quietly shrank the window. Now End is a fraction of the whole recording (clamped to what's
  left after Start), and a short window keeps its size while you move it through the sample with Start.
- **Start and End are precise.** A slow turn moves 2 ms of audio per detent, whatever the loop's length.
  In 0.9.1 each detent was 0.6 % of the loop: 270 ms on a 45 s loop. Turning faster accelerates
  smoothly, so one quick twist still crosses the whole loop. Positions are kept to the sample through
  saves and reloads.
- **Scrubbing Start is click-free.** The loop window glides (about 7.5 ms) to each new Start / End
  instead of jumping a detent at a time, so even a tiny loop can be scrubbed through the sample.
- **Smoother jumps.** Every jump inside a loop (Scatter, Seed slices, Jump mode, un-pause, a head
  turning on, jog moves, the Jump button) now crossfades over 5.8 ms instead of 1.45 ms, which could
  tick on bass. Ping-pong heads turn around exactly instead of snapping to the edge. Seed never cuts a
  slice under 256 samples, so very short windows use fewer slices.

### Undo
- **32 steps.** Undo walks back through recent gestures, newest first:
  - overdubs;
  - clears, and a recording started over an existing loop;
  - Dynamic overwrites;
  - randomise and reset.
- **One press brings back a whole gesture.** A gesture that touched all sixteen pads (Rnd All, Reset
  All, Sessions → Clear) comes back in one press, but uses 16 of the 32 steps. Clears and overwrites
  come back instantly.
- **Not undoable:** knob moves, and the first recording onto an empty pad.
- **Loading a session starts a fresh history**, so Undo can't put the last session's audio on its pads.
- **Messages:** `Undo: T3 overdub`, `Undo: T3 audio`, `Undo: T3 settings`, `Undo: 16 pads`, or
  `nothing to undo`. Clearing a pad says `T3 cleared (Undo restores)`.
- **Undo + pad no longer overdubs.** **Shift + Undo + pad** resets that loop's settings to factory
  (Wear healed, audio untouched), at once and without asking.

### Punch-in FX
- **In SYNC:**
  - **Loops, Chop, Reverse and Glide** play the live audio until the next grid line, then repeat the
    bit that just played, on the grid.
  - Their Rate / Len knob (and Strum's Rate) lands on note values, triplets included: 1/32 … 2 bars
    for Loops, Glide and Reverse; 1/4 … 1/32 for Strum.
  - **Pressure steps on the grid:**
    - Loops go 1 → ½ → ⅓ → ¼.
    - Reverse and Strum go through shorter note values.
    - Chop and Mosaic double.
    - A new step only takes over at the start of a full slice, so the repeats stay on the grid. A pad
      resting on a boundary doesn't flutter.
  - **Mosaic and Strum** fire their grains on the grid lines, with the live audio until the first.
  - The per-pad **LFO Rate** runs in note values (8 bars, 4, 2, 1 bar, 1/2, 1/4, 1/4T, 1/8, 1/8T,
    1/16, 1/16T, 1/32), locked to the bar.
  - Repeats are limited to 1 s of audio, so at slow tempos the longest values shorten.
  - A pitched repeat (Pit off centre) or a Glide runs at its own rate and leaves the grid.
- **In ASYNC, pressure is more fluid:**
  - It comes in quickly (about 15 ms) and blooms out slowly (about 65 ms) as you lift.
  - Chop and Mosaic now tighten smoothly with pressure (up to double speed) instead of jumping at half
    pressure.
  - Strum's strokes are humanised, a little early or late, and looser as you press. Its octave creeps
    in stroke by stroke from 40 % pressure instead of switching on at 70 %.
- **Readouts:** Chop and Mosaic show their grid as a note value (1/4 … 1/32). The LFO Rate shows in Hz
  in ASYNC (0.05–20 Hz) and as a note value in SYNC.

### Input (track button 1)
- **Input FX and Input Tape are one menu, on two pages:** *Input Tape* (the machine) and *Input EQ*.
  The layout is in *Where things moved*.
- **The Input EQ is a blend** of the Studer 962 presence band and the Tascam 424 MkII/MkIII channel EQ
  (from the owner's manual). Every sweep is logarithmic, and the knobs read 0.00–1.00:

  | Knob | Range |
  |---|---|
  | Low | ±15 dB shelf |
  | LowF (new) | 40–400 Hz, default 100 Hz (it was fixed at 120 Hz) |
  | Mid | ±12 dB peak (was ±11) |
  | MidF | 150 Hz – 7 kHz |
  | High | ±15 dB shelf |
  | HiF | 3–15 kHz, default 10 kHz |
  | LoCut | 20–800 Hz |
- **Generations** (the Gen knob, up to four passes) now re-applies the machine's head bump on every
  pass (on the machines that have one) and adds a touch of wow per pass, as well as the high-frequency
  loss.

### Sessions (≡ button)
- **Sessions live in the Move's UserLibrary,** in `/data/UserData/UserLibrary/Loopex`. The Schwung
  Manager's file browser shows them there, next to Magnéto's recordings. Each session is a folder
  `sessionNN` with:
  - `sNN_loop01.wav` … `sNN_loop16.wav`;
  - `sNN_wearNN.bin` for each worn loop;
  - `state.txt`, `meta.txt` and `name.txt`.

  Two-digit numbers keep them sorted. (Thanks Hannes.)
- **Clear** (K5) asks `CLEAR ALL LOOPS?`. It wipes all sixteen loops and resets their settings (the
  global pages stay), and the header goes back to *New*. One Undo brings it all back.
- **Reset** (K6) asks `RESET PAD N?`. It puts every setting of the selected loop back to factory and
  heals its Wear; the audio stays and keeps playing. **RstAll** (K7, `RESET ALL PADS?`) does it for all
  sixteen. "Factory" is exactly what a fresh Loopex starts with. (Thanks Hannes.)
- **Mode** (K8): ASYNC / SYNC, above.
- **In every question, K5 = NO and K8 = YES.** After a question opens or is answered, the knobs pause
  until they have all been still for 0.4 s. The rest of a twist used to land on NO (so Clear cancelled
  itself) or on the page underneath (flipping Mode).

### Screen, lights and knobs
- **Shift + pad and Loop + pad readouts stay while you hold the button:** `Speed | T3 = 1/2x`,
  `Length | T3 = 0.5x`. The pop-up messages `T3 speed 1/2x` / `T3 length 0.5x` stay about 2.5 s instead
  of vanishing under the footer. Speeds read `1/2x` (was `0.5x`). (Thanks djd_oz.)
- **Pad colours follow the real speed.** After a Reset, Undo, randomise or session load the pad
  recolours; speeds off the ½× / 1× / 2× steps show as 1× green.
- **Trigger knobs fire once per twist,** not once per detent: Perform → Jump and Scan, Save, Load,
  Delete, the FX sequencer's Clear, and the new ones. In 0.9.1 one twist of Jump fired four or five
  times.
- **The track-button lights come back** after visiting Schwung. Going back into Schwung and then into
  Loopex used to leave them dark until pressed.
- **No more first-turn jumps.** Right after opening Loopex, a knob could read as its minimum before the
  engine answered, so the first turn of End jumped from 100 % to 0 % (or Speed to −2 octaves). Knobs
  now keep their value and ask again.
- **The Capture light** is steady while the Dynamic menu is open and blinks while the sampler listens.

### Performance
- **Pitched loops hold their pitch under load.** The pitch shifters run on two background workers that
  now have twice the deadline and go ahead of Move's other background work. With 12 pitched loops, each
  used to slip back to its unshifted pitch for a few milliseconds about once a second; now it's a
  handful of single-block misses a minute. Engaging Pitch fades in about 12 ms later (you won't hear
  it; the playheads compensate).
- **Rnd All is light.** Randomising sixteen loops used to push the CPU meter to about 90 %; it now
  stays around 55 %.

Changed in 0.9.0, never announced:
- **Overdub was moved from a double-tap to Undo + pad** in 0.9.0. The double-tap made every pad press
  wait for a possible second one. *In 0.9.2 overdub moves again, to the step-button hold (see Loops).*
- **Tapping a recorded pad:** the first tap plays, the second **stops** it, and the third starts it
  again **from the Start point**, re-phasing all four playheads.

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
