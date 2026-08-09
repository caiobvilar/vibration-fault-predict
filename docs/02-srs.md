# Software/System Requirements Specification — vibration-fault-predict

| | |
|---|---|
| Project | vibration-fault-predict |
| Version | 1.0 |
| Status | draft |

This SRS is forward-looking: the project has no code yet (scaffolded state).
All requirements are in draft status and will be baselined at G1 review.

## 1. Purpose and scope

The vibration-fault-predict project shall perform on-device fault/anomaly
detection from a vibration signature, running a quantized model on the
F411-DISCO's IMU stream, distinguishing healthy from degraded rotating-machinery
operation.

## 2. Stakeholders and needs

| Need | Stakeholder | Need text |
|---|---|---|
| N-01 | Owner | A fixed-point vibration feature extractor + binary anomaly classifier running continuously on the F411-DISCO IMU stream. |
| N-02 | Owner | End-to-end latency per window <= 50 ms at the configured sample rate. |
| N-03 | Owner | Host-testable domain logic via port interfaces (no hardware includes in domain). |

## 3. Definitions and abbreviations

- **L1 gate** — Host (native) unit-test gate.
- **Vibration window** — A fixed-duration window of accelerometer samples (e.g., 1 kHz, 1024 samples).
- **Time-domain features** — RMS, kurtosis, crest factor computed per window.
- **Anomaly decision** — Binary healthy/degraded with confidence score.

## 4. System context

Board: F411-DISCO (STM32F411VET6, board MB1115 B-02). IMU: LSM6DS3TR-C on SPI.
The machine under test (rotating-machinery rig) does not exist yet — see
hardware blocker in PLAN.md. L1/L2 will use recorded or public vibration data;
L3/L4 require the rig.

## 5. Assumptions and constraints

- Streaming signal problem: windowed features feeding a classifier continuously.
- Reuses `gesture-imu`'s IMU driver port interface (i_clock, i_spi).
- L3/L4 blocked on rig availability (see PLAN.md).

## 6. Requirements

### 6.1 Functional

1. **VFP-FUN-001** (shall) — The vibration feature extractor shall compute a
   configurable set of time-domain features (RMS, kurtosis, crest factor) from
   each window of accelerometer data.
2. **VFP-FUN-002** (shall) — The anomaly classifier shall output a binary
   decision (healthy / degraded) with a confidence score from the extracted
   feature vector.

### 6.2 Performance

3. **VFP-PER-001** (shall) — The feature extraction plus classification latency
   per window shall not exceed 50 ms at the configured sample rate.

### 6.3 Interface

4. **VFP-INT-001** (shall) — The feature extractor and classifier shall
   communicate with the IMU driver through the port interface without
   hardware-specific code.

### 6.4 Constraints

5. **VFP-CON-001** (should) — The domain code shall compile on the host without
   any hardware or vendor includes.

## 7. Verification summary

| Method | Count |
|---|---|
| Test | 3 |
| Analysis | 2 |
| Inspection | 0 |
| Demonstration | 0 |

*Full traceability is in the generated `docs/06-rtm.md`.*

## 8. Open issues

| ID | Issue | Owner | Target |
|---|---|---|---|
| OI-01 | Machine-under-test vibration rig does not exist; L3/L4 blocked until built. | Owner | before L3 |
| OI-02 | Window size, sample rate, and feature set to be finalized in ADR. | Owner | before G1 |
| OI-03 | Source of public vibration datasets for L1/L2 validation to be identified. | Owner | before L1 |

## 9. Change log

| Version | Date | Change | Commit |
|---|---|---|---|
| 1.0 | 2026-08-09 | Initial draft baseline. | |