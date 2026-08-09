# PLAN — vibration-fault-predict

> **Migrated 2026-08-09** from the workbench harness repo into the portfolio
> meta-repo. Now `projects/T2-dsp/vibration-fault-predict`, following the
> program's reference repository layout (`03-toolchain.md §2`). Gate numbers
> use the program's four test levels (L1 unit, L2 SITL, L3 HIL, L4 acceptance)
> from `03-toolchain.md §3`.

Durable state across context windows. Read this first; update it before you
stop. If a future session cannot resume from this file alone, it is not
detailed enough.

## Goal

On-device fault/anomaly detection from vibration signature, running a
quantized model on `f411-disco`'s IMU stream, distinguishing healthy from
degraded rotating-machinery operation. This is the third leg of the
EdgeAI/TinyML trio: `tinyml-m0` proves fitting a model under a hard memory
budget, `gesture-imu` proves live sensor-driven classification, this one
proves the industrial predictive-maintenance framing that maps most directly
to the CV's "edge-AI deployment… 10+ models… latency reduction" line —
anomaly/fault detection is the canonical production use case for that claim.

Placed in **T2** (DSP): the core deliverable is the signal-processing pipeline
— windowed FFT or time-domain features (RMS, kurtosis, crest factor) feeding a
small classifier — the P08/P09 neighbourhood of
[10-T2-dsp.md](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/10-T2-dsp.md).

**This project has a real hardware blocker, stated honestly up front:** the
machine under test — a rotating-machinery vibration rig — **does not exist
yet**. It is not built and not on any bench. L1/L2 can proceed against
recorded or public vibration data; L3 (HIL) and L4 (acceptance) cannot start
until the rig exists. The program's P15 (BLDC FOC drive,
[12-T4-hardware.md §P15](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/12-T4-hardware.md)) is the natural future
source of a real motor rig to mount the IMU on.

## Target

- Sensing/compute board: `f411-disco` (mounted on or near the machine under
  test — this is the board that gets flashed, not the machine itself). See
  `docs/hardware/stm32f411e-disco.md`.
- Machine under test: a motor/vibration rig — **does not exist yet**, see
  blocker above.
- Highest gate reached: **none — scaffolded, no code yet**
- Constraint that drives the design: this is a streaming-signal problem
  (windowed FFT or time-domain features feeding the classifier), not a
  single-shot classification like `tinyml-m0`/`gesture-imu` — latency budget
  is per-window, continuously, not per-inference-on-demand.

## Steps

Tick as gates pass, not as code is written. Gates per [02-process.md](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/02-process.md),
test levels per [03-toolchain.md §3](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/03-toolchain.md).

- [ ] **SRS + G1.** EARS requirements (feature pipeline, detection accuracy,
      per-window latency, false-positive bound) before code. Seed
      `docs/02-srs.md` + `docs/requirements/`.
- [ ] Decide feature pipeline: raw time-domain windows vs. on-device FFT vs.
      hand-engineered features (RMS, kurtosis, crest factor) feeding a small
      classifier — record as an ADR, this is a real design choice with a
      RAM/latency/accuracy tradeoff, not a default.
- [ ] Source a stand-in dataset for L1/L2 development (e.g. a public
      bearing-fault vibration dataset) — label clearly in the README as a
      substitute for real bench data, not real bench data.
- [ ] L0 builds cross + host
- [ ] Static analysis clean, deviations recorded in `docs/adr/`
- [ ] L1 host unit tests against the stand-in dataset
- [ ] L2 SITL (Renode) — replay recorded/stand-in vibration data, selftest
      passes in emulation
- [ ] L3 HIL — **blocked.** Needs a real motor/vibration rig (P15's motor
      drive is the candidate source), with induced-fault fixtures (e.g. an
      imbalanced load) to generate real fault-condition data.
- [ ] L4 acceptance — blocked on L3.

## Decisions

Link ADRs. Do not restate them here.

- Toolchain: [docs/adr/0001-cubemx-cmake-toolchain.md](docs/adr/0001-cubemx-cmake-toolchain.md) applies as to every STM32
  project.
- Reuses the `f411-disco` IMU driver from `gesture-imu` once that project
  establishes it (`src/ports/` + `src/adapters/`) — do not fork a second
  copy. Part is confirmed: LSM303DLHC (U5) accelerometer over I2C1
  (SCL=PB6, SDA=PB9), per `docs/hardware/stm32f411e-disco.md` verified
  against board MB1115 B-02 (serial 217380895). Vibration sensing here
  almost certainly only needs the accelerometer half, not the gyro
  (L3GD20/U7) `gesture-imu` also brings up.
- Not yet decided: feature pipeline (see Steps above) and which public
  dataset stands in for real bench data during L1/L2.

## Open questions

Facts you need and do not have. Name the document and section. This is the
handoff list — the human answers these between sessions.

- [ ] Is a real vibration rig actually going to be built (e.g. on P15's motor
      drive)? If not, this project's L3/L4 need a different real motor/machine
      fixture named instead — the current plan assumes it is real near-term
      intent, not just a placeholder.
- [ ] LSM303DLHC electrical specs (ODR options at the ranges relevant to
      vibration sensing, noise floor) — pins/bus confirmed, datasheet not yet
      read.
- [ ] Which failure mode(s) to target first (imbalance, bearing wear,
      misalignment) — determines what an "induced fault" fixture at the bench
      actually needs to be.

## Log

Newest last. One line per session: what moved, what broke, where you stopped.

- `2026-08-09` — Migrated from workbench `projects/03-fault-predict` into
  `projects/T2-dsp/vibration-fault-predict` per the program layout. Content
  unchanged; the `inventory.yaml`-based blocker is restated as "no rig
  exists"; ADR + hardware sheet copied in; gate vocabulary updated.
- `2026-07-27` — IMU part (LSM303DLHC) confirmed via `gesture-imu`'s
  schematic check — no longer blocked on that question. Vibration-rig
  existence is still the open L3/L4 blocker.
- `2026-07-26` — Project scaffolded from `00-template` at the human's
  direction, as the third EdgeAI/TinyML project alongside `tinyml-m0` and
  `gesture-imu`. No code yet; flagged as blocked on hardware (no rig) for
  anything above L2.
