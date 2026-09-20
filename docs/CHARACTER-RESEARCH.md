# Character voicings — research record

Every number in Loopex's `MEQ_DEF` table, where it came from, and how far it can be
trusted. Written so that a future change can tell a documented figure from a judgement
call without re-doing the research.

**Read the tags strictly:**

| Tag | Meaning |
|---|---|
| **DOC** | Manufacturer manual, service manual, spec sheet or patent |
| **DOC-REISSUE** | Manufacturer documentation, but for a modern reissue standing in for vintage hardware |
| **MEASURED** | Independent bench measurement or peer-reviewed analysis |
| **COMM** | Forum, blog, dealer copy, magazine editorial |
| **CHOICE** | Our decision. Not a fact. Never present it as one. |
| **GAP** | Not findable. Deliberately left unmodelled rather than estimated. |

Plugin-emulation marketing is **not** hardware documentation. Where a figure exists only
in a plugin manual it is tagged COMM and named as such.

---

## Where Character sits

Char is the **output machine** — what the loops reach the world through. It is the last
coloured stage before the converter:

```
… → PUNCH FX → Drift → Pump → [ gSat → LoCut → HiCut → CHARACTER → Glue → Out → Limit ] → dither → hardware
```

Input Tape is the complementary stage: *media formats*, printed into the loop at record
time. Char is *named machines*, applied live at playback. Formats in, machines out.

## Table columns

`loHz loDb · midHz midDb midQ · hiHz hiDb · sat · crushHz · makeupDb · grit · bits · clip ·
asym · compand · bwHz · poles · glueAtk glueRel glueKnee glueDist · limCeil · thrust`

Notable ones: **grit** morphs the loop reader Hermite → linear → drop-sample. **bwHz** is
the reconstruction-filter ceiling. **poles** sets LoCut/HiCut order. **glueDist** is
distortion that rises *with* gain reduction. **thrust** is API's sidechain tilt.

---

## The thirteen voicings

### Off
Bypassed. `master_character` returns before touching anything.

### 962 — Studer 961/962 broadcast desk · *default*
Service manual supplied directly; the EQ was fitted by digitising the printed curve
family at 600 dpi and least-squares fitting RBJ shelves against it.

- Response **+0.5 / −1 dB**; −3 dB at **4.5 Hz and 45 kHz**; THD **<0.03 %** at +6 dBu;
  EIN ≤ −125 dBu — **DOC** (§1.7.2)
- Channel EQ is a **Fächerentzerrer** (fan equaliser). The published "20 Hz ±15 dB" and
  "20 kHz ±15 dB" are **measurement frequencies — the gain reached at the edge of the
  band — not shelf corners.** Reading them as corners is what made Bass and Treble
  inaudible for a year — **DOC**
- Fitted: low shelf **100 Hz S=0.64** (0.43 dB RMS), high shelf **4750 Hz S=0.50**
  (0.32 dB RMS) — **MEASURED** (our fit to their plot)
- Presence filter **Q = 1, 150 Hz–7 kHz, ±11 dB** — **DOC**
- Master limiter: ratio to **1:20**, threshold **+6 dBu**, stereo LINK, **PDM/VCA** — **DOC**
- Voiced as near-nothing, because the desk is transparent by design.

### Air — Focusrite ISA 110
The EQ Rupert Neve designed in 1985 for the Focusrite Studio Console at AIR Studios
Montserrat — hence the name, and hence the nickname its top end still carries.

- Parametric bands **40–400 Hz** and **600 Hz–6 kHz** (×3 ranges) — **DOC**
- HPF **36/60/105/185/330 Hz** and LPF **3.9/5.6/8.2/12/16 kHz**, both **18 dB/oct** — **DOC**
- THD **0.003 %** line, **0.0008 %** mic — **DOC**
- Transformer-coupled mic in, line in and output — **DOC**
- Shelf frequency points **never published by Focusrite**. Mix and SOS reviews give
  LF 33/56/95/160/270 and HF 3.3/4.7/6.8/10/15/18 kHz and disagree with each other —
  **COMM**. Our 95 Hz / 15 kHz carry that caveat.
- **Correction:** we first used 16 kHz for the HF shelf, having confused the unit's
  **low-pass** positions with its shelf.

### SSL — SSL 4000 bus
- E vs G is a documented *behavioural* difference: **E holds bandwidth constant with
  gain; G varies it** (more boost/cut → more selectivity) — **DOC** (X-Rack manual)
- '242 vs '02: ±18 dB vs ±15 dB, 18 dB/oct HPF — **DOC**, though SSL's own documents
  conflict on shelf slope (6 vs 12 dB/oct)
- Original-console band ranges exist only in licensed plugin manuals — **COMM**
- G-series bus compressor: **3 ms / 0.3 s / 4:1** — **DOC**
- No original-console THD or transformer data exists anywhere — **GAP**

### Neve — Neve 1073
- LF shelf **35/60/110/220 Hz ±16 dB**; mid bell **360/700/1.6k/3.2k/4.8k/7.2 kHz
  ±18 dB**; HF shelf **fixed 12 kHz ±16 dB**; HPF **50/80/160/300 Hz, 18 dB/oct** — **DOC**
- THD ≤0.07 % at +20 dBu — **DOC**
- **Third-harmonic dominant**, bench-measured — **MEASURED** (Sound On Sound, 1073N)
- Mid-band Q: AMS Neve state only "fixed Q". No number exists — **GAP**
- Dynamics from the **33609** (the 2254 diode bridge as a stereo bus box): comp attack
  3/6 ms, release 100–1500 ms plus Auto1 100 ms/2 s and Auto2 50 ms/5 s, ratio 1.5–6:1,
  soft knee reaching **full ratio ~6 dB above threshold** — **DOC-REISSUE**
- **THD per mode: <0.075 % bypassed, <0.2 % compressing, <0.45 % limiting** — **DOC**.
  The only unit in this entire table whose maker publishes distortion under gain
  reduction, and the sole basis for `glueDist`.

> **Trap:** a "0.3 % THD at 6 dB GR, third-harmonic dominant" figure circulates near 2254
> discussions. It is from an SOS review of the **Rupert Neve Designs 5254**, a different
> modern unit. Do not apply it here.

### Trident — Trident A-Range
Weakest-documented row in the table. Treat every figure with suspicion.

- LF shelf 50/80/100/150 Hz; low-mid 250/500/1k/2k; high-mid 3k/5k/7k/9k; HF shelf
  8k/10k/12k/15k; ±15 dB; **Q 1.3** — **DOC** for the *reissue*; Trident's own handbook
  for the original console prints no dB range and no Q
- The two sources **disagree on the lowest LF point** (50 vs 80 Hz)
- Transformer-coupled in and out; inductor mid bands — **DOC**
- **Harmonic order: GAP.** Do not assume 2nd or 3rd. Our asym is a **CHOICE**.
- Dynamics from the **CB9146** (FET): attack 20 µs–200 ms, release 25–500 ms, ratio to
  20:1 — **COMM only**, and specifically a forum user reading a photograph of a front
  panel. **No manufacturer documentation for this unit exists.**
- **Correction:** the CB9066 is an **equaliser, not a compressor**. That pairing was wrong.

### Studer — Studer A800 tape machine
Distinct from the 962 desk. Different machine, different character.

- 60 Hz–18 kHz ±1 dB at 15 ips; 60 Hz–20 kHz at 30 ips — **DOC**
- W&F **0.04 %** DIN peak weighted; operating level 510 nWb/m; bias 240 kHz — **DOC**
- **THD specified explicitly as 3rd harmonic, 1 % max** at operating level. Studer
  publish **no even-order figure at all** — **DOC**
- **Head bump: not documented by Studer.** The word does not appear in 1.1 MB of service
  text — **GAP**. Our ~60 Hz comes from Jack Endino's measured A80 MkII and A827
  siblings (+1 to +1.8 dB, 30–62 Hz at 15 ips) — **MEASURED, third party**
- The structurally useful law: the bump is a **damped ripple, not one peak**, and it
  shifts **one octave up per speed doubling** — **MEASURED**

### API — API 550A, with API 2500 dynamics
- LF **30/40/50/100/200/300/400 Hz**; MF **200/400/600/800/1500/3000/5000**; HF
  **2500/5000/7000/10k/12.5k/15k/20k**; ±12 dB stepped; **12 dB/oct** — **DOC**
- Proportional Q published **only as a graph**. The two community figures disagree by
  ~2× — **GAP**, deliberately not modelled
- **Correction:** we first used 900 Hz and 6 kHz. Neither is on either switch.
- Dynamics from the **2500**: attack .03/.1/.3/1/3/10/30 ms; release .05–2 s plus
  variable 50 ms–3 s, **no auto**; ratio 1.5–∞; HARD/MED/SOFT knee; TYPE OLD =
  feed-back, NEW = feed-forward; THAT 2180 VCAs with a 2252 RMS detector — **DOC**
- **THRUST** (US Patent 5,170,437): a sidechain filter **before** the RMS detector,
  **+10 dB/decade**, −15 dB at 20 Hz to +15 dB at 20 kHz, pivot ≈ **632 Hz**, inverse of
  the pink-noise curve. Bass therefore stops driving the gain reduction — **DOC**, with a
  plotted curve in API's manual and a five-stage RC ladder in the patent. We implement a
  first-order tilt, an approximation of that ladder.
- Max output +32 dB, response ±0.5 dB 20 Hz–50 kHz, S/N 120 dB, clips +28 dBu — **DOC**
- **Knee width in dB: GAP.** API's HARD/MED/SOFT plot has no scale on either axis and
  nobody has measured it. Ours is a **CHOICE**.
- **THD under gain reduction: GAP.** No measurement exists publicly, anywhere. A THAT-2180
  VCA bus compressor is meant to stay clean, so `glueDist = 0.00` is a **finding**, not an
  omission.
- **Caveat on the pairing itself:** 550A is 1967, 2500 is 2000s. This is matched by
  *function* — the API bus compressor people actually use — not by era, unlike
  Neve→33609 or ISA 110→ISA 130 which are genuine contemporaries.

### Ampex — Ampex ATR-102
- ±0.75 dB **200 Hz–20 kHz** at 30 ips; ±2 dB **35 Hz–28 kHz**. At 15 ips, ±0.75 dB
  100 Hz–15 kHz, ±2 dB 20 Hz–20 kHz — **DOC** (Table 1-4)
- **3rd harmonic <0.3 % at 370 nWb/m, <3.0 % at 1040 nWb/m (+9 VU). Even-order <0.1 %
  at reference** — **DOC**. That 1:3 ratio is what sets its asym, and the 0.3 → 3 %
  progression over +9 dB is why it has the lowest Limit ceiling.
- W&F ±0.03 % peak weighted; bias 432 kHz — **DOC**
- HF −3 dB point: **GAP** — Ampex publish only ±0.75 and ±2 dB windows
- Head bump: **GAP**, same as the A800. Ampex's spec structure *implies* a bound of
  ≤±2 dB below 200 Hz, but that is an inference from the table's shape, not a figure.
- **Ampex never made a compressor or limiter.** Catalogue OCR-searched; two circulating
  listings ("Ampex 350 + Triad opto", "AM-10 with Fairchild") are third-party racks.

### MPC60 — Akai MPC60
- **40 kHz fixed**; response **20 Hz–18 kHz**; **"12 bit sample resolution, special
  non-linear format, reduced noise"**; tuning +½/−1 octave — **DOC** (service manual §I)
- 16-bit ADC and DAC with companded 12-bit storage — **DOC** (operator's manual)
- **The companding law is never named.** "Special non-linear format" is a deliberate
  non-disclosure, not an omission — **GAP**. µ-law (µ=255) is our **CHOICE**: period-
  correct and behaves as described.
- It has a **real reconstruction filter**, unlike the E-mus. Its character is the
  companding quantiser, not imaging — hence low `grit` and `bwHz = 18000`.
- Service manual parts list uses internal Akai part numbers (`EF-…`), so filter component
  values are not derivable from it — **GAP**

### S950 — Akai S950
- Sample rate **7.5–48 kHz**; bandwidth **3–19.2 kHz** — **DOC**
- **`fc = fs × 0.4`**, annotated on the voice block diagram. Rate and bandwidth are
  **locked at 2.5:1** — **DOC**
- **6-pole, 36 dB/oct MF6CN-50** switched-capacitor filter on **both** record and
  playback (IC7 + IC109 ×8), plus a fixed 2-pole 18 kHz stage — **DOC**
- **12-bit linear. No companding** — **DOC**. Do not port the MPC60 assumption across.
- Pitch by **variable sample clock** — no interpolation, no drop-sample — **DOC**
- Frequency response, S/N, THD: **never published by Akai** — **GAP**
- **Correction:** independently adjustable bandwidth vs sample rate is a feature of
  **Inphonik's RX950 plugin**, not the hardware. We recommended this unit on a feature it
  does not have.

### SP-1200 — E-mu SP-1200
- **26.04 kHz** fixed (shared with the Drumulator and SP-12); **12-bit linear**, no
  companding — **DOC**
- **No reconstruction filter.** Output is bare zero-order hold; the imaging *is* the
  sound — **DOC**
- **Drop-sample pitch shifting**: fractional index, truncated, no interpolation. Pitching
  down repeats samples outright — **MEASURED** (Yeh, Nolting & Smith, CCRMA/ICMC 2007)
- No dedicated audio ADC — a 12-bit SAR loop around the playback DAC (AD7541) — **DOC**
- Output channel filters: ch 1–2 dynamic SSM2044, ch 3–6 fixed, **ch 7–8 unfiltered** — **DOC**
- Exact fixed-filter cutoffs: **GAP**

> The archive.org SP-1200 service manual scan is **image-only**; an early automated fetch
> returned confident "quotes" that were actually SP-12 text. Treat any online claim of
> quoting it with suspicion.

### Emu SP-12 — E-mu SP-12
- **Same 26.04 kHz clock, same 12-bit linear format, same absence of a reconstruction
  filter** as the SP-1200. The published "27.5 kHz" is a spec-sheet error; measured at
  26 kHz — **MEASURED** (CCRMA)
- Differentiated from the SP-1200 by its **darker fixed output filters**, which is what
  our `hiHz/hiDb` carries
- Anti-alias filter measured as needing an **order-11** digital fit, attenuating above
  ~15 kHz — **MEASURED**
- The manual's "42 dB per octave" is in a *generic tutorial sentence* — it is **not** a
  spec for this unit and is commonly misquoted
- SSM2044 self-oscillation trim **1.3 kHz** — **DOC**

---

## Era-matched dynamics

Most of these consoles have no compressor. Each voicing borrows the unit the same maker
built in the same era, rather than being given invented timings.

| Voicing | Unit | Topology | Pairing |
|---|---|---|---|
| Neve | 33609 | diode bridge | **Solid** — only source publishing THD under GR |
| Air | ISA 130 | VCA | **Solid** — same ISA rack, full spec table |
| SSL | G-series bus comp | VCA | **Solid** |
| 962 | its own desk limiter | PDM/VCA | **Solid** — in-unit |
| API | 2500 | VCA + RMS | **By function, not era** |
| Trident | CB9146 | FET | **Weak** — zero manufacturer documentation |
| Studer, Ampex | none exists | — | class-appropriate timings |
| all four samplers | none exists | — | neutral Glue, hard rail on Limit |

---

## What the research overturned

1. **"Transformers and tape mean even harmonics" is false.** Contradicted three times
   independently: SOS measured the Neve 1073 third-harmonic dominant; Studer specify the
   A800's THD *as* 3rd and publish no even-order figure; Ampex document the ATR-102 at
   <0.3 % 3rd against <0.1 % even-order. The `asym` column was built on folklore and is
   no longer used as a differentiator.
2. **"Juno chorus-console" was not a real machine** — and the voicing contained no chorus.
3. **The S950's independent bandwidth control** is a plugin feature, not Akai's.
4. **CB9066 is an EQ**, not a compressor.
5. **Crush-rate corrections were sonically inert.** `eqCrush` is an integer hold count, so
   MPC60 never crushed at 30 kHz *or* 40 kHz, and 22 kHz and 26.04 kHz both give ÷2.
6. **Nobody publishes harmonic order behind a THD figure** — Neve's 33609 excepted.

## Standing gaps

Head bump for both tape machines · Neve mid Q · MPC60 companding law · API knee width ·
API 2500 THD under GR · S950 frequency response and THD · SP-1200 fixed filter cutoffs ·
Trident's entire spec · harmonic order for every unit except the Neve.

## Sources

Studer 961/962 service manual (supplied) · [Studer A800 service notes](https://archive.org/details/Studer_A800_service_notes) ·
[Ampex ATR-100 manual](https://www.manualslib.com/manual/4157555/Ampex-Atr-100-Series.html) ·
[MRL equalization fundamentals](https://mrltapes.com/pages/equalization-fundamentals) ·
[Jack Endino, measured recorder curves](https://www.endino.com/graphs/) ·
[AMS Neve 1073](https://www.ams-neve.com/outboard/1073-range/1073-mic-preamp-equaliser/) ·
[AMS Neve 33609](https://www.ams-neve.com/outboard/limiter-compressors/33609-stereo-compressor/) ·
[SOS Neve 1073N](https://www.soundonsound.com/reviews/neve-1073n) ·
[Focusrite ISA110 manual](https://device.report/m/40d4d0dc7f71caeec2a5199e370bd3ec89f599b21a1ed0004dfe21e18eaaceaf.pdf) ·
[Focusrite ISA 130/131](https://fael-downloads-prod.focusrite.com/customer/prod/s3fs-public/downloads/isa-130-131-user-guide1.pdf) ·
[SSL X-Rack EQ](https://support.download.solidstatelogic.com/Legacy/User%20Manuals/82S6XR0B0B%20-%20X-Rack%20EQ.pdf) ·
[SSL Origin](https://www.solidstatelogic.com/assets/uploads/downloads/origin/SSL_Origin_User_Guide_V1-1_screen.pdf) ·
[API 550A](https://www.barryrudolph.com/recall/manuals/api550a.pdf) ·
[API 2500 manual, THRUST + knee plots](https://apiaudio.com/docs/manuals/2500_user_18-06-18.pdf) ·
[US Patent 5,170,437](https://patents.google.com/patent/US5170437A/en) ·
[THAT 2252 datasheet](https://thatcorp.com/datashts/THAT_2252_Datasheet.pdf) ·
[Trident A-Range handbook](https://tridentaudiodevelopments.com/wp-content/uploads/2021/02/a_range_user_guide_4.pdf) ·
[Akai MPC60 service manual](https://www.polynominal.com/akai-mpc60/akai-mpc60-service-manual.pdf) ·
[Akai S950 operator's manual](https://manuals.fdiskc.com/flat/Akai%20S-950%20Owners%20Manual.pdf) ·
[E-mu SP-12 service manual](https://www.vintagesynthparts.com/wp-content/uploads/2016/08/EMU_SP12_Service_Manual.pdf) ·
[Yeh, Nolting & Smith, SP-12 modelling, CCRMA 2007](https://ccrma.stanford.edu/~dtyeh/papers/yeh07_icmc_sp12.pdf)
