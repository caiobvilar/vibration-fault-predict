# vibration-fault-predict — on-device vibration anomaly detection

**On-device fault/anomaly detection from vibration signature on an STM32F411,
distinguishing healthy from degraded rotating-machinery operation — scaffolded;
the signal-processing pipeline (windowed FFT or time-domain features) is the
design decision in front, and the hardware rig it needs does not exist yet.**

The industrial framing of the EdgeAI trio: not digit or gesture classification,
but predictive maintenance — the canonical production use case for the "edge-AI
deployment" claim. The DSP substance lives in T2: windowed signal features
(RMS, kurtosis, crest factor) or an on-device FFT feeding a small quantized
classifier, with a per-window, continuous latency budget rather than
per-inference.

> **Status: migrated scaffold.** No code yet. The vibration source (a motor
> rig) does not exist; L1/L2 can proceed on a clearly-labelled public dataset,
> L3/L4 need the real rig. The program's P15 (BLDC FOC drive) is the natural
> future source of that rig.

## What this demonstrates (planned)

| | |
|---|---|
| **Signal processing** | Windowed FFT / time-domain features → small classifier; per-window latency |
| **Firmware** | Reuses `gesture-imu`'s IMU driver (LSM303DLHC, I2C1); quantized inference |
| **Process** | Feature-pipeline ADR, stand-in dataset labelled honestly, fault-injection framing |

## Verified facts in hand

| Fact | Value | Source |
|---|---|---|
| Accel/mag | LSM303DLHC (U5), I2C1: SCL=PB6, SDA=PB9 | MB1115 B.2 schematic sheet 6/6 |
| Board identity | `MB1115 B-02`, serial 217380895 | silkscreen, 2026-07-27 |

Full sheet with provenance in [docs/hardware/stm32f411e-disco.md](docs/hardware/stm32f411e-disco.md).

## The honest blocker

The machine under test does not exist. The plan is that P15's motor drive
(`12-T4-hardware.md §P15`) eventually becomes the vibration rig — the IMU
board mounts on the motor, induced-fault fixtures (e.g. an imbalanced load)
generate real fault-condition data. Until then, everything above L2 is
explicitly blocked, and the README/PLAN say so rather than pretending.

## Architecture (planned)

```
LSM303DLHC (I2C1) ──► src/domain/ window → features (FFT / RMS / kurtosis)
                                          │
                                          ▼
                               quantized classifier → fault flag
                                          │
                                          ▼
                          debug UART  ?peek vibration_fault
```

## Repository layout

| Path | Contents |
|---|---|
| `src/domain/` | Feature extraction + classifier logic — hardware-free, host-testable |
| `src/ports/` + `src/adapters/` | Reuses `gesture-imu`'s IMU driver |
| `test/unit/` | Unity tests against the stand-in dataset |
| `test/integration/` | Renode, replaying recorded/stand-in vibration data |
| `docs/adr/` | Toolchain decisions (0001, 0003) |
| `docs/hardware/` | STM32F411E-DISCO fact sheet |

## Build and run

Not yet buildable — no CMake project exists. SRS + G1 first, then the
feature-pipeline ADR, then the stand-in dataset.

## Documentation

- [PLAN.md](PLAN.md) — durable state, the blocker, open questions, log
- [docs/hardware/stm32f411e-disco.md](docs/hardware/stm32f411e-disco.md) — board facts

## What I'd do differently

- The vibration rig was assumed, then parked. The honest move was to name
  P15 as the source of real fault data from the start and treat everything
  above L2 as gated on it — that is now written into the plan, but the
  scaffold exists *because* the assumption was made without a commit behind it.
- A stand-in dataset is a substitute and will be labelled as one; the risk is
  that a model tuned on public bearing data looks too good. The real
  acceptance number has to come from the induced-fault bench campaign.

## License

Code: **Apache-2.0** (`LICENSE`) · Documentation: **CC BY 4.0**
(`docs/LICENSE-docs.md`) — per the program's publishing rules
([06-publishing.md §2.3](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/06-publishing.md)).
