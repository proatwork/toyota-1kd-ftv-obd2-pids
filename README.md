# Prado J150 — every value the ECUs expose through the ELM327

2013 Land Cruiser Prado J150, 1KD-FTV / A750F. Engine ECU `7E0`→`7E8`, transmission ECU `7E1`→`7E9`,
CAN 11-bit 500 kbps. Generated from `mode21_map.csv`, the two scans in `scans/` and
`MODE21_DIFF_idle1_vs_idlerpm.md`. **Off** = ignition on, engine off (`scans/20261105_000211_idle1`);
**Idle** = cold idle ~890 rpm (`scans/20261105_000624_idlerpm`).

**Reading the tables**

- **Bytes** count from the first byte after the `41 xx` / `61 xx` reply header. `8.2` = byte 8, bit 2,
  bits numbered MSB-first (bit 0 = 0x80); `0–1` = 16-bit big-endian.
- **Scaling**: value = raw × factor ± offset. Enum rows list the states.
- **Certainty**
  - **High** — standard OBD-II PID, or Techstream's own definition in a block pinned by a value that
    fits only that block, and the readings make sense.
  - **Med-high** — as High, but something is still unchecked (noted in *Basis*).
  - **Med** — definition exists but the readings look wrong, or a best guess with decent evidence.
  - **Low** — guess, or a block mapping we can't confirm.
- A value flagged ⚠ was outside Techstream's own min/max in that scan.

## 1. Mode 01 — standard OBD-II PIDs

Request `01xx` (append a reply-count digit, `01491`, for speed). Decoded per SAE J1979, except MAP.

### Engine (7E8)

| PID | Name | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|
| `0101` | Monitor status | MIL off, 0 DTC(s) | MIL off, 0 DTC(s) | High | standard PID |
| `0104` | Engine load | 0.0 % | 20.8 % | High | standard PID |
| `0105` | Coolant temp | 16 °C | 20 °C | High | standard PID |
| `010B` | Intake manifold pressure (MAP) | 101.6 kPa | 99.1 kPa | High | this ECU scales it ×64/51 kPa, not 1 kPa/bit — engine-off value matches baro (`0133`) |
| `010C` | Engine RPM | 0.0 rpm | 897.5 rpm | High | standard PID |
| `010D` | Vehicle speed | 0 km/h | 0 km/h | High | standard PID |
| `010F` | Intake air temp | 22 °C | 19 °C | High | standard PID |
| `0110` | MAF air flow | 0.4 g/s | 10.82 g/s | High | standard PID |
| `0111` | Throttle position | 70.2 % | 25.5 % | High | on this diesel it's the intake throttle valve, not the pedal |
| `011C` | OBD standard | EOBD | EOBD | High | standard PID |
| `011F` | Run time since engine start | 0 s | 50 s | High | standard PID |
| `0121` | Distance with MIL on | 0 km | 0 km | High | standard PID |
| `0124` | PID 24 | DB EA 00 00 | DB EA 00 00 | High | standard PID (A/F sensor equivalence ratio + voltage); not decoded here — see `2124` |
| `012C` | Commanded EGR | 0.0 % | 38.8 % | High | standard PID |
| `012D` | EGR error | 99.2 % | 0.0 % | High | standard PID |
| `0130` | Warm-ups since codes cleared | 33 | 33 | High | standard PID |
| `0131` | Distance since codes cleared | 1150 km | 1150 km | High | standard PID |
| `0133` | Barometric pressure | 101 kPa | 101 kPa | High | standard PID |
| `0134` | PID 34 | DB EA 80 00 | DB EA 80 00 | High | standard PID (A/F sensor equivalence ratio + current); not decoded here — see `2134` |
| `0142` | Control module voltage | 12.402 V | 13.867 V | High | standard PID |
| `0149` | Accelerator pedal position D | — | 15.7 % | High | the real accelerator pedal; % is sensor voltage, rest ≈ 15.7 % |
| `014A` | Accelerator pedal position E | — | 31.8 % | High | standard PID |
| `014C` | Commanded throttle actuator | — | 41.2 % | High | standard PID |
| `014D` | Time run with MIL on | 0 min | 0 min | High | standard PID |
| `014E` | Time since codes cleared | 2265 min | 2269 min | High | standard PID |
| `014F` | PID 4F | — | 08 00 00 20 | High | standard PID (maximum values for λ, sensor voltage/current, MAP); not decoded here |

### Transmission (7E9)

| PID | Name | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|
| `0101` | Monitor status | MIL off, 0 DTC(s) | MIL off, 0 DTC(s) | High | standard PID |
| `0104` | Engine load | 0.0 % | 21.2 % | High | standard PID |
| `0105` | Coolant temp | 16 °C | 20 °C | High | standard PID |
| `010C` | Engine RPM | 0.0 rpm | 897.5 rpm | High | standard PID |
| `010D` | Vehicle speed | 0 km/h | 0 km/h | High | standard PID |
| `011C` | OBD standard | EOBD | EOBD | High | standard PID |
| `011F` | Run time since engine start | 0 s | 50 s | High | standard PID |
| `0121` | Distance with MIL on | 0 km | 0 km | High | standard PID |
| `0130` | Warm-ups since codes cleared | 34 | 34 | High | standard PID |
| `0131` | Distance since codes cleared | 1159 km | 1159 km | High | standard PID |
| `0142` | Control module voltage | 12.128 V | 13.476 V | High | standard PID |
| `014D` | Time run with MIL on | 0 min | 0 min | High | standard PID |
| `014E` | Time since codes cleared | 2300 min | 2304 min | High | standard PID |

## 2. Mode 21 bundles of Mode 01 values (engine)

Not in Techstream's list; decoded by matching the Mode 01 scan byte for byte. One request returns
many Mode 01 values — the gauge firmware polls `21B6`.

| Block | Bytes | Name | Scaling | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| `21B6` | 0–3 | Monitor status (PID 01) | bitfield | 00 0E 80 00 | 00 0E 80 00 | High | matches PID 01 |
| `21B6` | 4–5 | Fuel system status (PID 03) | bitfield | 00 00 | 00 00 | High | matches PID 03 |
| `21B6` | 6 | Engine load (PID 04) | raw × 100/255 % | 0.0 % | 16.9 % | High | matches PID 04 |
| `21B6` | 7 | Coolant (PID 05) | raw − 40 °C | 16 °C | 26 °C | High | matches PID 05 |
| `21B6` | 8 | MAP (PID 0B) | raw × 64/51 kPa | 101.6 kPa | 97.9 kPa | High | ×64/51 per Techstream's `210B`; engine-off value = baro |
| `21B6` | 9–10 | Engine RPM (PID 0C) | raw ÷ 4 rpm | 0 rpm | 872 rpm | High | matches PID 0C |
| `21B6` | 11 | Vehicle speed (PID 0D) | raw km/h | 0 km/h | 0 km/h | High | matches PID 0D |
| `21B6` | 12 | Intake air temp (PID 0F) | raw − 40 °C | 23 °C | 19 °C | High | matches PID 0F |
| `21B6` | 13–14 | MAF (PID 10) | raw ÷ 100 g/s | 0.40 g/s | 8.98 g/s | High | matches PID 10 |
| `21B6` | 15 | Intake throttle (PID 11) | raw × 100/255 % | 70.2 % | 22.0 % | High | matches PID 11 |
| `21B7` | 0 | OBD standard (PID 1C) | enum | EOBD | EOBD | High | matches PID 1C |
| `21B7` | 1–2 | Run time (PID 1F) | raw s | 0 s | 111 s | High | matches PID 1F |
| `21B7` | 3–4 | Distance with MIL on (PID 21) | raw km | 0 km | 0 km | High | matches PID 21 |
| `21B7` | 5–6 | Fuel rail pressure (PID 23) | raw × 10 kPa | 0.0 bar | 374.7 bar | High | matches PID 23 |
| `21B7` | 7–10 | PID 24 (A/F sensor λ + voltage) | — | DB EA 00 00 | DB EA 00 00 | Med | same bytes as PID 24; not decoded |
| `21B8` | 0 | Commanded EGR (PID 2C) | raw × 100/255 % | 0.0 % | 40.0 % | High | matches PID 2C |
| `21B8` | 1 | EGR error (PID 2D) | (raw − 128) × 100/128 % | 99.2 % | 0.0 % | High | matches PID 2D |
| `21B8` | 2 | Warm-ups since codes cleared (PID 30) | raw | 33 | 33 | High | matches PID 30 |
| `21B8` | 3–4 | Distance since codes cleared (PID 31) | raw km | 1150 km | 1150 km | High | matches PID 31 |
| `21B8` | 5 | Barometric pressure (PID 33) | raw kPa | 101 kPa | 101 kPa | High | matches PID 33 |
| `21B8` | 6–9 | PID 34 (A/F sensor λ + current) | — | DB EA 80 00 | DB EA 80 00 | Med | same bytes as PID 34; not decoded |
| `21B9` | 0–1 | Control module voltage (PID 42) | raw mV | 12.25 V | 13.95 V | High | matches PID 42 |
| `21BA` | 0 | Accelerator pedal D (PID 49) | raw × 100/255 % | 15.7 % | 15.7 % | High | matches PID 49 |
| `21BA` | 1 | Accelerator pedal E (PID 4A) | raw × 100/255 % | 31.8 % | 31.8 % | High | matches PID 4A |
| `21BA` | 2 | Commanded throttle actuator (PID 4C) | raw × 100/255 % | 60.4 % | 40.4 % | High | matches PID 4C |
| `21BA` | 3–4 | Time run with MIL on (PID 4D) | raw min | 0 min | 0 min | High | matches PID 4D |
| `21BA` | 5–6 | Time since codes cleared (PID 4E) | raw min | 2266 min | 2270 min | High | matches PID 4E |

## 3. Mode 21 — engine (7E0), Techstream definitions

All 186 parameters Techstream's engine Data List defines, mapped to Mode 21 blocks by `tse_map.py`.
Send `ATSH7E0` first, then `21xx`. Blocks `2101`–`214E` mirror Mode 01 PIDs (same bytes as `01xx`).

### `2101` — block pinned by: 4 B; MIL + DTC count = Mode 01 PID 01 layout

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0.0 | MIL | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; same layout as the Mode 01 PID |
| 0.1–7 | Number of Emission DTC | raw |  | 0 | 0 | High | Techstream definition; same layout as the Mode 01 PID |
| 1.2 | Fuel System Monitor Result | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 1.5 | Complete Parts Monitor | 0=Not Avl, 1=Avail |  | Avail | Avail | High | Techstream definition; same layout as the Mode 01 PID |
| 1.6 | Fuel System Monitor | 0=Not Avl, 1=Avail |  | Avail | Avail | High | Techstream definition; same layout as the Mode 01 PID |
| 1.7 | Misfire Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.0 | EGR/VVT Monitor | 0=Not Avl, 1=Avail |  | Avail | Avail | High | Techstream definition; same layout as the Mode 01 PID |
| 2.1 | O2S(A/FS) Heater Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.2 | O2S(A/FS) Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.3 | A/C Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.4 | 2nd Air Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.5 | EVAP Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.6 | Heated Catalyst Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 2.7 | Catalyst Monitor | 0=Not Avl, 1=Avail |  | Not Avl | Not Avl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.0 | EGR/VVT Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.1 | O2S(A/FS) Heater Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.2 | O2S(A/FS) Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.3 | A/C Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.4 | 2nd Air Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.5 | EVAP Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.6 | Heated Catalyst Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |
| 3.7 | Catalyst Monitor | 0=Compl, 1=Incmpl |  | Compl | Compl | High | Techstream definition; same layout as the Mode 01 PID |

### `2104` — block pinned by: PID 04

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Calculate Load | raw × 0.392157 | % | 0.0 | 20.4 | High | Techstream definition; same layout as the Mode 01 PID |

### `2105` — block pinned by: PID 05

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Coolant Temp | raw − 40 | °C | 16 | 21 | High | Techstream definition; same layout as the Mode 01 PID |

### `210B` — block pinned by: PID 0B

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | MAP | raw × 1.2549 | kPa | 102 | 98 | High | Techstream definition; same layout as the Mode 01 PID |

### `210C` — block pinned by: PID 0C

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Engine Speed | raw × 0.25 | rpm | 0 | 893 | High | Techstream definition; same layout as the Mode 01 PID |

### `210D` — block pinned by: PID 0D

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Vehicle Speed | raw | km/h | 0 | — | High | Techstream definition; same layout as the Mode 01 PID |

### `210F` — block pinned by: PID 0F

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Intake Air | raw − 40 | °C | 22 | 19 | High | Techstream definition; same layout as the Mode 01 PID |

### `2110` — block pinned by: PID 10

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | MAF | raw × 0.01 | gm/s | — | 10.21 | High | Techstream definition; same layout as the Mode 01 PID |

### `2111` — block pinned by: PID 11

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Throttle Sensor Volt % | raw × 0.392157 | % | 70.2 | 25.1 | High | Techstream definition; same layout as the Mode 01 PID |

### `211C` — block pinned by: PID 1C

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | OBD Requirements | 1=OBD II (California ARB), 2=OBD (US Federal EPA), 3=OBD and OBD II, 4=OBD I, 5=Not OBD compliant, 6=EOBD (Euro OBD), 7=EOBD and OBD II, 8=EOBD and OBD, 9=EOBD, OBD and OBD II, 10=JOBD (Japan OBD), 11=JOBD and OBD II, 12=JOBD and EOBD, 13=JOBD, EOBD, and OBD II, 14=EURO 4, 15=EURO 5, 17=Engine Manufacturer Diagnostics (EMD), 18=Engine Manufacturer Diagnostics Enhanced (EMD+), 19=Heavy Duty On-Board Diagnostics (Child/Partial), 20=Heavy Duty On-Board Diagnostics, 21=World Wide Harmonized OBD, 23=Heavy Duty Euro OBD Stage I without NOx control, 24=Heavy Duty Euro OBD Stage I with NOx control, 25=Heavy Duty Euro OBD Stage II without NOx control, 26=Heavy Duty Euro OBD Stage II with NOx control, 28=Brazil OBD Phase 1, 29=Brazil OBD Phase 2, 30=Korean OBD, 31=India OBD I, 32=India OBD II, 33=Heavy Duty Euro OBD Stage VI, 34=OBD, OBD II and HD OBD, 35=Brazil OBD Phase 3, 36=Motorcycle, Euro OBD-I, 37=Motorcycle, Euro OBD-II, 38=Motorcycle, China OBD-I, 39=Motorcycle, Taiwan OBD-I, 40=Motorcycle, Japan OBD-I, 41=China Nationwide Stage 6, 42=Brazil OBD Diesel |  | EOBD (Euro OBD) | EOBD (Euro OBD) | High | Techstream definition; same layout as the Mode 01 PID |

### `211F` — block pinned by: PID 1F

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Engine Run Time | raw | s | 0 | 62 | High | Techstream definition; same layout as the Mode 01 PID |

### `2121` — block pinned by: PID 21

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | MIL ON Run Distance | raw | Km | 0 | 0 | High | Techstream definition; same layout as the Mode 01 PID |

### `2123` — block pinned by: PID 23

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Fuel Press | raw × 10 | kPa | 0 | 39570 | High | Techstream definition; same layout as the Mode 01 PID |

### `2124` — block pinned by: 4 B; lambda + voltage = Mode 01 PID 24 layout

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | AF Lambda B1S1 | raw × 0.000122072 |  | 6.872 ⚠ | 6.872 ⚠ | Med | Techstream definition, but a reading fell outside its own range |
| 2–3 | AFS Voltage B1S1 | raw × 0.000122 | V | 0.00 | 0.00 | High | Techstream definition; same layout as the Mode 01 PID |

### `212C` — block pinned by: PID 2C

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Target EGR Position | raw × 0.392157 | % | 0.0 | 39.2 | High | Techstream definition; same layout as the Mode 01 PID |

### `2130` — block pinned by: PID 30

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Warmup Cycle Cleared DTC | raw |  | 33 | 33 | High | Techstream definition; same layout as the Mode 01 PID |

### `2131` — block pinned by: PID 31

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Distance from DTC Cleared | raw | km | 1150 | 1150 | High | Techstream definition; same layout as the Mode 01 PID |

### `2133` — block pinned by: PID 33

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Atmosphere Pressure | raw | kPa | 101 | 101 | High | Techstream definition; same layout as the Mode 01 PID |

### `2134` — block pinned by: 4 B; AFS current = Mode 01 PID 34 layout

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 2–3 | AFS Current B1S1 | raw × 0.00390625 − 128 | mA | 0.00 | 0.00 | High | Techstream definition; same layout as the Mode 01 PID |

### `2142` — block pinned by: PID 42

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Battery Voltage | raw × 0.001 | V | — | 14.0 | High | Techstream definition; same layout as the Mode 01 PID |

### `2149` — block pinned by: PID 49

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Accel Sens. No.1 Volt % | raw × 0.392157 | % | 15.7 | 20.4 | High | Techstream definition; same layout as the Mode 01 PID |

### `214A` — block pinned by: PID 4A

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Accel Sens. No.2 Volt % | raw × 0.392157 | % | — | 35.3 | High | Techstream definition; same layout as the Mode 01 PID |

### `214C` — block pinned by: PID 4C

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Throttle Motor DUTY | raw × 0.392157 | % | 60.4 | 51.8 | High | Techstream definition; same layout as the Mode 01 PID |

### `214D` — block pinned by: PID 4D

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Running Time from MIL ON | raw | min | 0 | 0 | High | Techstream definition; same layout as the Mode 01 PID |

### `214E` — block pinned by: PID 4E

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Time after DTC Cleared | raw | min | 2265 | 2270 | High | Techstream definition; same layout as the Mode 01 PID |

### `2178` — block pinned by: 9 B; three exhaust temps, supported bits 0x07 = S1-S3

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0.4 | Exhaust Temperature B1S4 Supported | 0=Unsupp, 1=Supp |  | Unsupp | Unsupp | High | Techstream definition; block pinned, readings plausible |
| 0.5 | Exhaust Temperature B1S3 Supported | 0=Unsupp, 1=Supp |  | Supp | Supp | High | Techstream definition; block pinned, readings plausible |
| 0.6 | Exhaust Temperature B1S2 Supported | 0=Unsupp, 1=Supp |  | Supp | Supp | High | Techstream definition; block pinned, readings plausible |
| 0.7 | Exhaust Temperature B1S1 Supported | 0=Unsupp, 1=Supp |  | Supp | Supp | High | Techstream definition; block pinned, readings plausible |
| 1–2 | Exhaust Temperature B1S1 | raw × 0.1 − 40 | °C | 11.8 | 67.5 | High | Techstream definition; block pinned, readings plausible |
| 3–4 | Exhaust Temperature B1S2 | raw × 0.1 − 40 | °C | 11.8 | 61.2 | High | Techstream definition; block pinned, readings plausible |
| 5–6 | Exhaust Temperature B1S3 | raw × 0.1 − 40 | °C | 3.1 | 21.2 | High | Techstream definition; block pinned, readings plausible |
| 7–8 | Exhaust Temperature B1S4 | raw × 0.1 − 40 | °C | -40.0 | -40.0 | High | Techstream definition; block pinned, readings plausible |

### `21A3` — block pinned by: 16 B; target idle 875 rpm, 1 start, 894 rpm starter-off at idle

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Engine Start Time | raw × 1048.58 | ms | 0 | 0 | High | Techstream definition; block pinned, readings plausible |
| 1 | Engine Speed (Starter Off) | raw × 6.25 | rpm | 0 | 894 | High | Techstream definition; block pinned, readings plausible |
| 2 | Starter Count | raw |  | 0 | 1 | High | Techstream definition; block pinned, readings plausible |
| 3 | Run Dist of Previous Trip | raw × 1.28 | km | 1 | 0 | High | Techstream definition; block pinned, readings plausible |
| 4 | Electric Duty Feedback Value | raw × 0.15625 | mm3/st | 2.3 | 3.8 | High | Techstream definition; block pinned, readings plausible |
| 5 | A/C Duty Feedback Value | raw × 0.15625 | mm3/st | 0.0 | 0.0 | High | Techstream definition; block pinned, readings plausible |
| 6 | PS Duty Feedback Value | raw × 0.15625 | mm3/st | 0.0 | 0.0 | High | Techstream definition; block pinned, readings plausible |
| 7 | Idle Injection Volume (Min) | raw × 0.15625 | mm3/st | 0.0 | 0.0 | High | Techstream definition; block pinned, readings plausible |
| 9.2 | Immobiliser Fuel Cut History | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 9.3 | MAF Low | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 9.4 | Boost Pressure Low | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 9.5 | Common Rail Pressure Low | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 9.6 | Engine Coolant Temp High | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 10–11 | Target Idle Engine Speed | raw × 0.78125 | rpm | 904 | 875 | High | Techstream definition; block pinned, readings plausible |
| 12.4 | Rough Idle #4 | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 12.5 | Rough Idle #3 | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 12.6 | Rough Idle #2 | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 12.7 | Rough Idle #1 | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 13 | IG-ON Time | raw × 131.072 | ms | 0 | 33423 | High | Techstream definition; block pinned, readings plausible |
| 14 | Glow Request Lighting Time | raw × 131.072 | ms | 786 | 786 | High | Techstream definition; block pinned, readings plausible |
| 15 | MAF/Estimate MAF Ratio | raw × 0.01 |  | 0.00 | 0.44 | High | Techstream definition; block pinned, readings plausible |

### `21AD` — block pinned by: 29 B; injector feedback #1-4 at bytes 17-20 centred on 0x40

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Pilot 1 Injection Period | raw | µs | 0 | 221 | High | Techstream definition; block pinned, readings plausible |
| 2–3 | Pilot 2 Injection Period | raw | µs | 0 | 237 | High | Techstream definition; block pinned, readings plausible |
| 4–5 | Main Injection Period | raw | µs | 0 | 356 | High | Techstream definition; block pinned, readings plausible |
| 6–7 | After Injection Period | raw | µs | 0 | 0 | High | Techstream definition; block pinned, readings plausible |
| 8–9 | Pilot 1 Injection Timing | raw × 0.1 (signed) | deg(CA) | 0.0 | -14.7 | High | Techstream definition; block pinned, readings plausible |
| 10–11 | Pilot 2 Injection Timing | raw × 0.1 (signed) | deg(CA) | 0.0 | -9.9 | High | Techstream definition; block pinned, readings plausible |
| 12–13 | Main Injection Timing | raw × 0.1 − 90 | deg(CA) | -3.0 | -4.2 | High | Techstream definition; block pinned, readings plausible |
| 14–15 | After Injection Timing | raw × 0.1 (signed) | deg(CA) | 0.0 | 21.4 | High | Techstream definition; block pinned, readings plausible |
| 16 | Inj Vol Feedback Learning | raw × 0.078125 − 10 | mm3/st | 0.0 | 0.0 | High | Techstream definition; block pinned, readings plausible |
| 17 | Injection Feedback Val #1 | raw × 0.15625 − 10 | mm3/st | 0.0 | -0.3 | High | Techstream definition; block pinned, readings plausible |
| 18 | Injection Feedback Val #2 | raw × 0.15625 − 10 | mm3/st | 0.0 | -0.2 | High | Techstream definition; block pinned, readings plausible |
| 19 | Injection Feedback Val #3 | raw × 0.15625 − 10 | mm3/st | 0.0 | 0.2 | High | Techstream definition; block pinned, readings plausible |
| 20 | Injection Feedback Val #4 | raw × 0.15625 − 10 | mm3/st | 0.0 | 0.0 | High | Techstream definition; block pinned, readings plausible |
| 21–22 | Injection Volume | raw × 0.0195312 | mm3/st | 0.00 | 9.36 | High | Techstream definition; block pinned, readings plausible |
| 23–24 | Pump SCV Learning Value | raw × 0.125 (signed) | mA | 35.2 | 35.2 | High | Techstream definition; block pinned, readings plausible |

### `21B5` — block pinned by: 6 B; ATF temp from ECT 18.8 C, gear 0 in P

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Received MIL from ECT | 0=OFF, 90=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 1 | Shift Position Sig from ECT | 0=1st, 1=2nd, 2=3rd, 3=4th, 4=5th, 5=6th, 6=7th, 7=8th |  | 1st | 1st | High | Techstream definition; block pinned, readings plausible |
| 2–3 | A/T Oil Temp from ECT | raw × 0.00390625 − 40 | °C | 17.5 | 18.8 | High | Techstream definition; block pinned, readings plausible |
| 4 | SPD (NO) | raw × 50 | rpm | 0 | 0 | High | Techstream definition; block pinned, readings plausible |
| 5.7 | ECT Lock Up | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |

### `21C1` — block pinned by: 18 B; ASCII KDJ15# 1KDFTV

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–6 | Model Code | ASCII text |  | KDJ15#  | KDJ15#  | High | Techstream definition; block pinned, readings plausible |
| 7–12 | Engine Type | ASCII text |  | 1KDFTV | 1KDFTV | High | Techstream definition; block pinned, readings plausible |
| 13 | Cylinder Number | raw |  | 4 ⚠ | 4 ⚠ | Med | Techstream definition, but a reading fell outside its own range |
| 14 | Transmission Type | 0=NA, 1=MT, 2=ECT 4th, 4=ECT 5th, 8=ECT 6th, 16=CVT, 32=MMT(5th), 64=MMT(6th), 128=ECT 8th |  | ECT 5th | ECT 5th | High | Techstream definition; block pinned, readings plausible |
| 15 | Destination | ASCII text |  | W | W | High | Techstream definition; block pinned, readings plausible |
| 16 | Model Year | raw + 1900 |  | 2011 ⚠ | 2011 ⚠ | Med | Techstream definition, but a reading fell outside its own range |
| 17 | System Identification | 1=Gasoline, 2=Diesel, 4=CNG/LPG, 5=Gasoline&CNG/LPG, 8=ECO, 9=ECO, 10=Diesel&ECO, 12=CNG/LPG&ECO, 16=MHV, 32=HV, 33=HV |  | Diesel | Diesel | High | Techstream definition; block pinned, readings plausible |

### `21D3` — block pinned by: 18 B; cruise switch bits, all 0 (C1 is the ASCII ID block)

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 8.2 | Cancel Switch | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 8.3 | SET/COAST Switch | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 8.4 | RES/ACC Switch | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |

### `21D9` — block pinned by: 3 B; A/T oil temp 20.0 C

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0.5 | Shift SW Status (R Range) | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 1–2 | A/T Oil Temperature 1 | raw × 0.00390625 − 40 | °C | 17.5 | 20.0 | High | Techstream definition; block pinned, readings plausible |

### `21DD` — block pinned by: 29 B; fuel temp byte 0 tracks coolant, target rail 378 bar vs actual 396

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | Fuel Temperature | raw − 40 | °C | 16 | 18 | High | Techstream definition; block pinned, readings plausible |
| 1 | Accel Position | raw × 0.390625 | % | 0.00 | 0.00 | High | Techstream definition; block pinned, readings plausible |
| 2 | Actual Throttle Position | raw (signed) | % | -1 | 85 | High | Techstream definition; block pinned, readings plausible |
| 3 | VN Turbo Command | raw (signed) | % | 71 | 94 | High | Techstream definition; block pinned, readings plausible |
| 4–5 | Target Pump SCV Current | raw × 0.125 | mA | 0.0 | 1035.9 | High | Techstream definition; block pinned, readings plausible |
| 6–7 | Inj. FB Vol. for Idle | raw × 0.00244141 (signed) | mm3/st | 0.00 | -5.37 ⚠ | Med | Techstream definition, but a reading fell outside its own range |
| 8.1 | Add. INJ Flow Excessive | 0=Normal, 1=Error |  | Normal | Normal | High | Techstream definition; block pinned, readings plausible |
| 8.2 | DPF Overtemperature | 0=Normal, 1=Error |  | Normal | Normal | High | Techstream definition; block pinned, readings plausible |
| 8.3 | DPF Thermal Deteriorate | 0=Normal, 1=Deteriorated |  | Normal | Normal | High | Techstream definition; block pinned, readings plausible |
| 8.4 | DPF No Activate | 0=Activate, 1=No Activate |  | Activate | Activate | High | Techstream definition; block pinned, readings plausible |
| 8.5 | DPF PM Block | 0=No Block, 1=Block |  | No Block | No Block | High | Techstream definition; block pinned, readings plausible |
| 8.6 | Catalyst Memory Error | 0=No Error, 1=Error |  | No Error | No Error | High | Techstream definition; block pinned, readings plausible |
| 8.7 | Injector Memory Error | 0=No Error, 1=Error |  | No Error | No Error | High | Techstream definition; block pinned, readings plausible |
| 9.6 | Reju Pilot Quantity Learning | 0=READY, 1=NG |  | READY | READY | High | Techstream definition; block pinned, readings plausible |
| 10.4–7 | Pilot Quantity Learning | 0=Standby, 1=Wait, 2=Learn, 4=Stop, 8=Comple |  | Standby | Standby | High | Techstream definition; block pinned, readings plausible |
| 11 | Fuel Return Temp | raw − 40 | °C | 40 | 41 | Med | reads 40–41 °C even cold with the engine off; sensor may not be fitted |
| 12 | Target Throttle Position | raw (signed) | % | 0 | 85 | High | Techstream definition; block pinned, readings plausible |
| 13–14 | Catalyst Differential Press | raw × 0.00012207 (signed) |  | 0.0000 | 0.0187 ⚠ | Med | sits outside Techstream's own range; a DPNR field, probably unused on this DPF engine |
| 15–16 | Exhaust Fuel Addition FB | raw × 0.0078125 (signed) |  | 1.00 | 1.00 | High | Techstream definition; block pinned, readings plausible |
| 17–18 | Target Booster Pressure | raw × 0.0195312 (signed) | kPa | 98.36 | 116.70 | High | Techstream definition; block pinned, readings plausible |
| 19–20 | Target Common Rail Pressure | raw × 10 | kPa | 30000 | 37830 | High | Techstream definition; block pinned, readings plausible |
| 21 | Actual VN Position | raw × 0.5 | % | 100.0 | 94.0 | High | Techstream definition; block pinned, readings plausible |
| 22–23 | VN Position Sensor Out | raw × 7.62939e-05 | V | 1.704 | 1.860 | High | Techstream definition; block pinned, readings plausible |
| 24 | VN Motor Duty | raw × 0.5 | % | 0.0 | 0.0 | High | Techstream definition; block pinned, readings plausible |
| 25–26 | VN Close Learn Value | raw × 7.62939e-05 | V | 1.646 | 1.646 | High | Techstream definition; block pinned, readings plausible |
| 27 | PM Accumulation Ratio | raw × 2 | % | 58 | 58 | Med-high | Techstream definition; idle doesn't move it, not yet seen under load or regen |
| 28 | Glow Control Unit Duty | raw × 0.5 | % | 0.0 | 63.5 | High | Techstream definition; block pinned, readings plausible |

### `21DE` — block pinned by: 5 B; accel sensor volts 0.8/1.6 V match pedal % 15.7/31.8

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Common Rail Pres Sens 2 | raw × 10 | kPa | 0 | 36570 | High | Techstream definition; block pinned, readings plausible |
| 2 | Accel Sensor Out No.1 | raw × 0.0195312 | V | 0.8 | 0.8 | High | Techstream definition; block pinned, readings plausible |
| 3 | Accel Sensor Out No.2 | raw × 0.0195312 | V | 1.6 | 1.6 | High | Techstream definition; block pinned, readings plausible |
| 4 | Alternate Duty Ratio | raw | % | 32 | 51 | High | Techstream definition; block pinned, readings plausible |

### `21DF` — block pinned by: 16 B; learned values static engine off vs idle

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Throttle Close Learning Val. | raw × 0.00762939 | deg | 17.4 | 17.4 | High | Techstream definition; block pinned, readings plausible |
| 2 | VN Turbo Type | 0=Not Avl, 1=Commo, 2=Vacuum, 3=CAN Com, 4=DC |  | DC | DC | High | Techstream definition; block pinned, readings plausible |
| 3 | VN Turbo Max Angle | raw × 0.390625 | % | 99.6 | 99.6 | High | Techstream definition; block pinned, readings plausible |
| 4 | VN Turbo Min Angle | raw × 0.390625 | % | 39.8 | 39.8 | High | Techstream definition; block pinned, readings plausible |
| 5–6 | Injection Pressure Correction | raw × 0.0195312 − 500 | mm3/st | 0.0 | 3.9 | High | Techstream definition; block pinned, readings plausible |
| 7–8 | AF Sensor Learning Value | raw × 0.000152588 | V | 1.86 | 1.86 | High | Techstream definition; block pinned, readings plausible |
| 9–10 | Diff. Press. Sensor Corr. | raw × 0.00390625 − 10 | kPa | -0.3 | -0.3 | High | Techstream definition; block pinned, readings plausible |
| 11 | DPNR/DPF Status Reju(PM) | 0=Standby, 1=Ready, 2=Operate, 3=Compl |  | Standby | Standby | Med-high | Techstream definition; only ever read Standby, a regen not yet seen |
| 12 | Idle Signal Output Value | raw |  | 0 | 1 | High | Techstream definition; block pinned, readings plausible |
| 13–14 | Boost Pressure Deviation | raw × 0.0195312 (signed) | kPa | -3.57 ⚠ | 18.42 | Med | Techstream definition, but a reading fell outside its own range |
| 15.7 | VN Turbo Operation Prohibit | 0=OK, 1=NG |  | OK | OK | High | Techstream definition; block pinned, readings plausible |

### `21E1` — block pinned by: 8 B; only 8-byte block

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0 | # Codes(Include History) | raw |  | 0 | 0 | Med-high | block pinned only by being the one 8-byte reply |

### `21E6` — block pinned by: 4 B; neutral switch on in P, all learn statuses OK

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0.0 | Starter Signal | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 0.2 | A/C Signal | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 0.3 | Neutral Position SW Signal | 0=OFF, 1=ON |  | ON | ON | High | Techstream definition; block pinned, readings plausible |
| 0.5 | Stop Light Switch | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 0.6 | Power Steering Signal | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 1.0 | Immobiliser Communication | 0=OFF, 1=ON |  | ON | ON | High | Techstream definition; block pinned, readings plausible |
| 2.4 | EGR Close Lrn. Status | 0=NG, 1=OK |  | OK | OK | High | Techstream definition; block pinned, readings plausible |
| 2.5 | Diesel Throttle Learn Status | 0=NG, 1=OK |  | OK | OK | High | Techstream definition; block pinned, readings plausible |
| 2.7 | Power Steer. Sig. Record | 0=OFF, 1=ON |  | ON | ON | High | Techstream definition; block pinned, readings plausible |
| 3.5 | VN Close Learn Status | 0=OFF, 1=ON |  | ON | ON | High | Techstream definition; block pinned, readings plausible |

### `21EA` — block pinned by: 6 B; actual EGR 40 % = commanded EGR 40 %

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | EGR Close Lrn. Val. | raw × 7.62939e-05 | V | 0.67 | 0.67 | High | Techstream definition; block pinned, readings plausible |
| 2 | Actual EGR Valve Pos. | raw × 0.392157 | % | 0.0 | 40.0 | High | Techstream definition; block pinned, readings plausible |
| 3.5 | EGR Cooler Bypass VSV2 | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 3.6 | EGR Cooler Bypass VSV | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 3.7 | EGR Operation Prohibit | 0=OK, 1=NG |  | OK | OK | High | Techstream definition; block pinned, readings plausible |
| 4 | EGR Motor Duty #1 | raw × 0.5 | % | 27.0 | 12.5 | High | Techstream definition; block pinned, readings plausible |
| 5 | EGR Cooler Bypass Position | 0=OFF, 1=Bypass, 2=Half Pos, 3=Cooler |  | Bypass | Bypass | High | Techstream definition; block pinned, readings plausible |

### `21EE` — block pinned by: 5 B; switch bits change with engine start

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0.0 | Swirl Control Valve VSV | 0=OFF, 1=ON |  | OFF | ON | High | Techstream definition; block pinned, readings plausible |
| 0.7 | ACT VSV | 0=OFF, 1=ON |  | ON | ON | High | Techstream definition; block pinned, readings plausible |
| 1.2 | Electric Fan Motor | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 2.5 | ACM Inhibit | 0=OFF, 1=ON |  | OFF | ON | High | Techstream definition; block pinned, readings plausible |
| 2.6 | TC and TE1 | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 3.5 | Pressure Discharge Valve | 0=OFF, 1=ON |  | OFF | OFF | High | Techstream definition; block pinned, readings plausible |
| 4.7 | Eng Oil Press Switch Valve | 0=OFF, 1=ON |  | OFF | ON | High | Techstream definition; block pinned, readings plausible |

### `21EF` — block pinned by: 4 B; oil pressure 2 kPa engine off -> 126 kPa idling

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | DPF Differential Pressure | raw × 0.00390625 − 5 | kPa | 0.301 | 0.426 | Med-high | Techstream definition; idle doesn't move it, not yet seen under load or regen |
| 2–3 | Engine Oil Pressure | raw × 0.078125 − 2560 | kPa | 1.953 | 126.406 | High | Techstream definition; block pinned, readings plausible |

### `21F1` — block pinned by: 5 B; initial coolant 16.9 C = coolant at key-on

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Initial Engine Coolant Temp | raw × 0.625 − 40 | °C | 16.9 | 16.9 | High | Techstream definition; block pinned, readings plausible |
| 2–3 | Initial Intake Air Temp | raw × 0.625 − 40 | °C | 23.1 | 24.4 | High | Techstream definition; block pinned, readings plausible |
| 4 | Intake Air Temp (Turbo) | raw − 40 | °C | 18 | 16 | High | Techstream definition; block pinned, readings plausible |

### `21F3` — block pinned by: 10 B; all FF (cylinder speeds only fill during an active test) - weakest link

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0–1 | Engine Speed of Cyl #1 | raw × 0.78125 | rpm | 51199 ⚠ | 51199 ⚠ | Low | block mapping is the weakest link: reads all FF outside Techstream's active test |
| 2–3 | Engine Speed of Cyl #2 | raw × 0.78125 | rpm | 51199 ⚠ | 51199 ⚠ | Low | block mapping is the weakest link: reads all FF outside Techstream's active test |
| 4–5 | Engine Speed of Cyl #3 | raw × 0.78125 | rpm | 51199 ⚠ | 51199 ⚠ | Low | block mapping is the weakest link: reads all FF outside Techstream's active test |
| 6–7 | Engine Speed of Cyl #4 | raw × 0.78125 | rpm | 51199 ⚠ | 51199 ⚠ | Low | block mapping is the weakest link: reads all FF outside Techstream's active test |
| 8–9 | Av Engine Speed of All Cyl | raw × 0.78125 | rpm | 51199 ⚠ | 51199 ⚠ | Low | block mapping is the weakest link: reads all FF outside Techstream's active test |

### `unmapped` — no block found

| Bytes | Name | Scaling | Unit | Off | Idle | Certainty | Basis |
|---|---|---|---|---|---|---|---|
| 0.0 | Check Mode | 0=OFF, 1=ON |  | — | — | Low | Techstream defines it, but no Mode 21 block matched |

## 4. Mode 21 — transmission (7E1), guesses

The saved Techstream session covers the engine only, so these come from the engine-off vs idle diff.
Saving a session with the ECT Data List open and running `tse_map.py` on it would settle them.
Send `ATSH7E1` first.

| Block | Bytes | Off | Idle | Candidate decode | Likely name | Certainty |
|---|---|---|---|---|---|---|
| `21D9` | 4–5 | 3A 20 | 3D 40 | raw ÷ 256 − 40 → 18.1 → 21.3 °C | ATF temp No.1 | Med-high |
| `21D9` | 6–7 | 3A 20 | 3C A0 | raw ÷ 256 − 40 → 18.1 → 20.6 °C | ATF temp No.2 (or torque-converter outlet) | Med |
| `21D9` | 0 | 00 | 10 | flag | Shift / lock-up status bit? | Low |
| `21B4` | 0–1 | 00 00 | 0D 51 | raw ÷ 4 → 852 rpm | Engine RPM (as the TCM sees it) | Med-high |
| `21B4` | 2 | 40 | 3B | raw − 40 → 19 °C | Intake air temp | Med |
| `21B4` | 3 | 38 | 47 | raw − 40 → 31 °C | Coolant | Med-high |
| `21B4` | 5–6 | 00 00 | 01 BB | 443 (×2 ≈ 886 rpm in P) | Turbine / input speed? | Low |
| `21E1` | 6–7 | 06 63 | 0F 09 | rises ~8.7/s, same as engine `21E1` | Timer since key-on (unit unknown) | High (that it's a timer) |

## 5. Blocks that answer but have no definition yet

These reply to `21xx` but nothing above decodes them. Raw idle bytes after `61 xx`.

| ECU | Block | Length | Idle bytes | Note |
|---|---|---|---|---|
| 7E0 | `2180` | 4 | `00 00 00 01` |  |
| 7E0 | `21A0` | 4 | `20 08 0F C1` |  |
| 7E0 | `21C0` | 4 | `C0 00 20 9F` |  |
| 7E0 | `21C2` | 12 | `36 30 54 32 30 00 00 03 04 00 00 00` | ID block, starts with ASCII “60T20” |
| 7E0 | `21DC` | 23 | `03 20 5C 00 00 00 0F 0F 0E 2D 83 01 80 7E 53 00 1B 5A 20 …` |  |
| 7E0 | `21E0` | 4 | `F6 46 F0 00` |  |
| 7E0 | `21E2` | 5 | `F6 F7 D9 F3 C0` |  |
| 7E0 | `21E3` | 1 | `00` |  |
| 7E0 | `21E4` | 1 | `04` |  |
| 7E0 | `21E7` | 4 | `02 2C 51 00` |  |
| 7E0 | `21F2` | 6 | `A2 52 3E 3E FF 00` |  |
| 7E0 | `21F4` | 4 | `00 00 00 00` |  |
| 7E1 | `2180` | 4 | `00 00 00 01` |  |
| 7E1 | `21A0` | 4 | `00 00 17 C1` |  |
| 7E1 | `21B6` | 11 | `00 00 00 00 00 00 00 38 00 00 00` | likely the TCM's own Mode 01 bundle (cf. engine section 2) |
| 7E1 | `21B7` | 5 | `06 00 C1 00 00` | likely the TCM's own Mode 01 bundle (cf. engine section 2) |
| 7E1 | `21B8` | 3 | `22 04 87` | likely the TCM's own Mode 01 bundle (cf. engine section 2) |
| 7E1 | `21B9` | 2 | `34 B8` | likely the TCM's own Mode 01 bundle (cf. engine section 2) |
| 7E1 | `21BA` | 4 | `00 00 09 03` | likely the TCM's own Mode 01 bundle (cf. engine section 2) |
| 7E1 | `21C0` | 4 | `C0 00 00 C1` |  |
| 7E1 | `21C1` | 18 | `4B 44 4A 31 32 23 20 41 37 35 30 46 20 04 04 57 6F 02` | ID block, starts with ASCII “KDJ12# A750F” |
| 7E1 | `21C2` | 11 | `36 30 34 34 30 00 00 03 04 00 00` | ID block, starts with ASCII “60440” |
| 7E1 | `21DA` | 2 | `01 04` |  |
| 7E1 | `21E0` | 4 | `E4 00 00 00` |  |
| 7E1 | `21E2` | 5 | `F6 F7 D9 F3 40` |  |
| 7E1 | `21E3` | 1 | `00` |  |
| 7E1 | `21E6` | 1 | `10` |  |

Blocks `2101`–`2160` on both ECUs mirror their Mode 01 PIDs (section 1) and are left out here; the
engine ones Techstream defines are in section 3.
