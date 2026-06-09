# QIDI I-Fast → OrcaSlicer Port Knowledge Base

Living reference for porting the **QIDI Tech I-Fast** into OrcaSlicer.

| Field | Value |
|---|---|
| **Branch** | `QIDI_I-Fast_Port` |
| **Repo** | [OrcaSlicer_QIDI_I-Fast_Port](.) (fork/port workspace) |
| **Status** | S3D-aligned profiles (needs hardware validation) |
| **Last updated** | 2026-06-08 |

---

## 1. Project goal

Add first-class **QIDI I-Fast** support to OrcaSlicer:

- Machine / extruder / process / filament presets under `resources/profiles/Qidi/`
- Correct dual-extruder behavior (auto-lift toolheads, PVA support workflows)
- Heated chamber and high-temp hotend settings
- Start/end G-code compatible with the **CBD** controller firmware
- G-code thumbnails in QIDI format (if the printer firmware expects them)
- Optional: WiFi send/print via the CBD UDP protocol (separate from newer Moonraker-based QIDI printers)

---

## 2. Printer overview

### Hardware (from QIDI sources)

| Spec | Value | Source |
|---|---|---|
| Build volume | **330 × 250 × 320 mm** | [Qidi-Print `i-fast.def.json`](https://github.com/QIDITECH/Qidi-Print/blob/main/resources/definitions/i-fast.def.json), [S3D profile](QIDI_I-Fast_S3D_Files/Qidi%20Technology%20i-fast%20Printer%20Profile.fff), [product page](https://eu.qidi3d.com/products/qidi-i-fast-a-pioneer-in-solving-complex-printing) |
| Extruders | **2** (auto-lift dual extruder) | All sources |
| Hotend max temp | **350 °C** (all-metal) | Qidi-Print definition `about` |
| Heated bed | Yes | Qidi-Print / S3D |
| Heated chamber | Yes, up to **~60 °C** | Product page; `machine_heated_build_volume: true` in Qidi-Print |
| Motion | Dual Z-axis, linear guides | Product page |
| Nozzle (default) | **0.4 mm** | S3D profile, Qidi-Print extruder defs |
| Filament | 1.75 mm | Qidi-Print base `qidi` definition |
| Controller board | **CBD** (not Klipper) | Qidi-Print `metadata.board: "CBD"` |
| Manufacturer series | `i-series` | Qidi-Print definition |
| Stock slicer | Qidi-Print (Cura-based) | [QIDITECH/Qidi-Print](https://github.com/QIDITECH/Qidi-Print) |

### Product highlights

From the [EU product page](https://eu.qidi3d.com/products/qidi-i-fast-a-pioneer-in-solving-complex-printing):

- Industrial frame, ~20% faster than prior generation, ~100 cc/h target
- Wide material support (PLA, ABS, PETG, Nylon, etc.) aided by heated chamber
- Dual extruder + PVA soluble support for complex parts
- Pre-installed dual extruder; chamber heat to 60 °C

### Support & documentation

- [QIDI I-Fast Wiki](https://wiki.qidi3d.com/en/I-Fast) — quick start, unboxing, motherboard diagram, troubleshooting
- After-sales: `Afast@qd3dprinter.com`, `Bfast@qd3dprinter.com`

---

## 3. Community demand

- [OrcaSlicer Issue #1784 — I-Fast](https://github.com/OrcaSlicer/OrcaSlicer/issues/1784)
  - Opened Aug 2023 requesting I-Fast in the printer list
  - Closed as **not planned** / **stale**
  - No linked PR or branch in upstream at time of writing
  - This port branch addresses that gap

---

## 4. Reference materials in this repo

### Simplify3D export (`QIDI_I-Fast_S3D_Files/`)

| File | Purpose |
|---|---|
| `Qidi Technology i-fast Printer Profile.fff` | Full machine + process settings (XML) |
| `Qidi I-Fast Factory File.factory` | S3D factory bundle (binary) |
| `readme.md` | Notes that these are S3D reference files for porting |

**Key values extracted from the S3D profile:**

| Setting | S3D value |
|---|---|
| Build volume | 330 × 250 × 320 mm |
| Homing | X/Y **min**, Z **max** |
| Nozzle diameter | 0.4 mm (both extruders) |
| Retraction | 1.5 mm @ 1800 mm/min (30 mm/s) per extruder |
| Max volumetric flow | 900 mm³/min per extruder |
| Default print speed | 3600 mm/min (60 mm/s) |
| Travel XY / Z | 6000 / 1000 mm/min |
| Accel XY / Z / E | 1000 / 150 / 3000 mm/s² |
| Jerk XY / Z / E | 600 / 24 / 300 mm/min |
| Thumbnail encoding | **Qidi Technology** (`qidi`), 300×300 |
| Tool-change retract | 12 mm @ 600 mm/min |
| Support material default | PVA |

**S3D start G-code (default / right extruder primary):**

```gcode
G28 ; home all axes
G1 X330 Y0 Z50 F3600 ; position for heating
M141 S[chamber0_temperature] ; set heated chamber temperature
M140 S[platform0_temperature] ; set build platform temperature
M190 S[platform0_temperature] ; stabilize build platform temperature
M104 S[extruder0_temperature] T0 ; set right extruder temperature
M109 S[extruder0_temperature] T0 ; stabilize right extruder temperature
```

**S3D end G-code:**

```gcode
G1 Z320 F2400 ; move away from finished print
G1 X360 Y250 F3600 ; move build platform to max XY
G1 Z320 F900 ; lower build plate
M104 S0 T0 ; turn off right extruder
M104 S0 T1 ; turn off left extruder
M140 S0 ; turn off heated build platform
M141 S0 ; turn off heated chamber
M107 ; turn off part cooling fan
M84 ; disable motors
```

**S3D tool-change (pre):**

```gcode
; IF new_tool == 0 → G1 X330 F[xy_travel_speed]
; IF new_tool == 1 → G1 X0  F[xy_travel_speed]
T[new_tool]
```

**S3D post-processing:** `{REPLACE "M106 S0" "M107"}`

---

## 5. Qidi-Print definition (primary upstream reference)

Source: [`resources/definitions/i-fast.def.json`](https://github.com/QIDITECH/Qidi-Print/blob/main/resources/definitions/i-fast.def.json)

Inherits: `qidi` → `fdmprinter`

| Override | Value |
|---|---|
| `machine_width` / `depth` / `height` | 330 / 250 / 320 |
| `machine_extruder_count` | 2 |
| `machine_heated_build_volume` | true |
| `cool_fan_*` | Not settable per extruder (shared part cooling) |
| `chamber_cooling_fan_speed` | enabled |
| `shutdown_after_printing` | enabled |
| `exclude_materials` | `qidi_abs_rapido`, `qidi_pla_rapido`, `qidi_pla_rapido_matte` |

**Qidi-Print start G-code** (includes dual-extruder prime lines):

```gcode
G28
G0 X0 Y0 Z50 F3600
M190 S{material_bed_temperature_layer_0}
M104 T0 S{material_print_temperature_layer_0, 0}
M109 T1 S{material_print_temperature_layer_0, 1}
M109 T0 S{material_print_temperature_layer_0, 0}
G0 X0 Y6 Z0.3 F3600
T1
G92 E-19
G1 X{machine_width} E0 F2400
T0
G92 E-19
G0 X{machine_width} Y4 F3600
G1 X5 E0 F2400
```

**Qidi-Print end G-code:**

```gcode
M104 S0 T0
M104 S0 T1
M140 S0
;Retract the filament
G92 E0
G1 E-3 F300
G0 Z{machine_height}
G0 X{machine_width} Y0 F3600
M84
```

**Extruder mapping:**

```json
"machine_extruder_trains": {
  "0": "i-fast_extruder_1",
  "1": "i-fast_extruder_2"
}
```

**UI / assets in Qidi-Print:**

- Platform model: `i-fast.stl`
- Platform offset: `[0, -0.3, 0]`

### Qidi base defaults (`qidi.def.json`)

Useful process defaults when creating Orca process presets:

- `gcode_flavor` equivalent: Marlin-style
- `speed_print`: 50 mm/s, `speed_travel`: 100 mm/s, `speed_layer_0`: 20 mm/s
- `switch_extruder_retraction_amount`: 10 mm
- `build_volume_temperature` max: 80 °C
- `material_print_temperature` max warning: 350 °C
- `retraction_hop`: 0.2 mm
- Chamber-aware cooling and support defaults

---

## 6. CBD WiFi protocol (Qidi-Print)

Source: [`python/qidi/Wifi/CBDConnect.py`](https://github.com/QIDITECH/Qidi-Print/blob/main/python/qidi/Wifi/CBDConnect.py)

The I-Fast uses QIDI's **CBD** board with a **UDP** networking stack (port **3000**), not Moonraker/Klipper.

### Connection

| Parameter | Value |
|---|---|
| Protocol | UDP |
| Port | **3000** |
| Discovery | Broadcast on port 3000 |
| Keepalive | `M4000` every 3 s; initial handshake `M4001` + `M99999` + `M20` |

### Status line format (parsed by `printer_info_update`)

Contains tokens such as:

`B:` (bed), `E1:` / `E2:` (extruders), `X:` `Y:` `Z:`, `F:` (fan), `D:` (print progress), `T:` (state), optional `I:` / `L:` (chamber/filament sensor)

### Custom / notable G-code commands

| Command | Purpose |
|---|---|
| `M4001` | Handshake / machine info (steps/mm, build size, encoding) |
| `M4000` | Poll / keepalive |
| `M99999` | Initial connect sequence |
| `M20` | List SD files |
| `M28 <file>` | Begin SD write |
| `M29` | End SD write |
| `M6030 ":<file>" I1` | Start print of uploaded file |
| `M6032` | Download file trigger |
| `M3000` / `M3001 I<n>` | SD file read chunks |
| `M22` | Close/release SD |
| `U100 '<name>'&<checksum>&` | Rename file on printer |

### File upload

- Optional **compressed** upload (`.gcode.tz`) via `VC_compress_gcode` helper
- Binary chunks with XOR checksum + `0x83` terminator
- Resend protocol: `resend <offset>` responses

### Implications for OrcaSlicer port

- Existing `QidiPrinterAgent` (Moonraker-based) in OrcaSlicer **does not** cover I-Fast WiFi
- I-Fast network printing would need a **CBD-specific agent** or users export G-code to USB/SD
- Phase 1 port can focus on **profiles + G-code**; network agent is a later phase

---

## 7. OrcaSlicer — existing QIDI support

### Vendor profiles (`resources/profiles/Qidi/`)

Current `Qidi.json` machine models (no I-Fast yet):

- Qidi Q1 Pro, Q2, Q2C
- Qidi X-CF Pro, X-Max, X-Max 3, X-Max 4
- Qidi X-Plus, X-Plus 3, X-Plus 4
- Qidi X-Smart 3

**Closest references for this port:**

| Printer | Relevance |
|---|---|
| **Qidi X-CF Pro** | Older enclosed QIDI, dual-extruder-style machine preset arrays, hardened nozzle |
| **Qidi X-Max** | Earlier X-series, similar brand defaults |
| **Qidi X-Max 3 / X-Plus 3** | Newer Klipper machines — good for profile *structure*, not firmware |

### Code touchpoints

| File | Relevance |
|---|---|
| `src/slic3r/Utils/QidiPrinterAgent.hpp/.cpp` | Moonraker agent for **newer** QIDI printers (AMS/filament sync) |
| `src/libslic3r/GCode/Thumbnails.cpp` | `thumbnail_QIDI` compressed thumbnail tag |
| `src/libslic3r/PresetBundle.cpp` | `VendorType::Klipper_Qidi` for Qidi vendor |
| `src/slic3r/GUI/CreatePresetsDialog.cpp` | Qidi model list in preset wizard (no I-Fast) |
| `src/libslic3r/PrintConfig.cpp` | Wipe tower notes mention Qidi + filament cutter |

### Thumbnails

S3D profile uses **Qidi thumbnail encoding** at 300×300. Orca already has `thumbnail_QIDI` support in `Thumbnails.cpp` — verify whether I-Fast firmware expects the same format when enabling printer thumbnails.

---

## 8. OrcaSlicer developer resources

Official wiki: [OrcaSlicer Wiki](https://www.orcaslicer.com/wiki/)

**Most relevant for this port:**

| Topic | Link |
|---|---|
| How to create profiles | [developer_reference/how_to_create_profiles](https://www.orcaslicer.com/developer_reference/how_to_create_profiles.html) |
| How to build | [developer_reference/how_to_build](https://www.orcaslicer.com/developer_reference/how_to_build.html) |
| How to run tests | [developer_reference/how_to_test](https://www.orcaslicer.com/developer_reference/how_to_test.html) |
| Preset / PresetBundle | [developer_reference/preset_and_bundle](https://www.orcaslicer.com/developer_reference/preset_and_bundle.html) |
| Built-in G-code placeholders | [developer_reference/built_in_placeholders_variables](https://www.orcaslicer.com/developer_reference/built_in_placeholders_variables.html) |
| Machine G-code settings | [printer_settings/machine gcode](https://www.orcaslicer.com/printer_settings/machine%20gcode/printer_machine_gcode.html) |
| Multimaterial / wipe tower | [printer_settings/multimaterial](https://www.orcaslicer.com/printer_settings/multimaterial/printer_multimaterial_setup.html) |
| How to download PR artifacts | [developer_reference/how_to_download_pr_artifacts](https://www.orcaslicer.com/developer_reference/how_to_download_pr_artifacts.html) |

### Build (from `AGENTS.md`)

```bash
# Windows (local)
cmake --build . --config RelWithDebInfo --target ALL_BUILD -- -m

# Windows (GitHub Actions — fork)
# Workflows: `.github/workflows/build_all.yml` (+ build_orca/deps/check_cache)
# 1. Push `QIDI_I-Fast_Port` with workflow files
# 2. GitHub → Settings → Actions → allow workflows
# 3. Actions → **Build all** → Run workflow → branch `QIDI_I-Fast_Port`
# 4. Download artifacts: `OrcaSlicer_Windows_*` (installer + portable zip)
# First run builds deps from scratch (~2–4 h); later runs use cache when `deps/**` unchanged.
```

---

## 9. Port work breakdown

### Phase 1 — Profiles (minimum viable)

- [x] Add `Qidi I-Fast` to `resources/profiles/Qidi.json` `machine_model_list`
- [x] Create `machine/Qidi I-Fast.json` model stub (bed model, textures, `model_id`, default materials)
- [x] Create `machine/Qidi I-Fast 0.4 nozzle.json`
  - [x] `printable_area` for 330×250
  - [x] `printable_height`: 320
  - [x] 2 extruders (`nozzle_diameter`: 0.4 + 0.4; offsets 0x0 pending hardware check)
  - [x] `gcode_flavor`: marlin
  - [x] Start/end G-code from Qidi-Print + S3D (`M141`/`M107` in end script)
  - [x] `toolchange_gcode` park at X330 / X0 per S3D
  - [x] QIDI thumbnails: `300x300/M4010` (S3D `M4010` chunked format, not `;gimage:` COLPIC)
  - [x] Heated chamber: `M141`/`M191` in start G-code; ABS 55°C / PA 50°C with `activate_chamber_temp_control`
  - [x] `support_chamber_temp_control` enabled on machine preset
  - [x] Conditional extruder heat/park in start G-code (`is_extruder_used`) — no unused nozzle heat or wipe
  - [x] `single_extruder_multi_material` = 0 (physical dual extruder; enables ooze shield when dual printing)
- [x] Create process presets (`0.30mm Fast`, `0.20mm Medium`, `0.10mm High` — S3D quality tiers)
- [x] Create filament presets (`Generic PLA/ABS/PETG/PVA/PA @Qidi I-Fast` from S3D materials)
- [x] Reuse global Qidi Generic filaments (no new filament JSON; Rapido excluded in model defaults)
- [x] Placeholder cover image `Qidi I-Fast_cover.png` (copied from X-CF Pro — replace with I-Fast art)
- [ ] Dedicated bed plate assets (`i-fast.stl` from Qidi-Print — currently reuses X-CF Pro visuals)
- [x] Update `CreatePresetsDialog.cpp` model list

#### S3D FFF → Orca profile mapping (2026-06-08)

| S3D (`Qidi Technology i-fast Printer Profile.fff`) | Orca preset |
|---|---|
| Build 330×250×320 | `printable_area` / `printable_height` in `Qidi I-Fast 0.4 nozzle.json` |
| Right + Left 0.4 mm extruders | `nozzle_diameter`: `["0.4","0.4"]` |
| Retraction 1.5 mm @ 1800 mm/min | `retraction_length` 1.5, `retraction_speed` 30 |
| Tool-change retract 12 mm @ 600 mm/min | `retract_length_toolchange` 12 |
| Default print 3600 mm/min (60 mm/s) | `fdm_process_qidi_ifast_common` speed overrides |
| First layer 35% → 21 mm/s | `initial_layer_speed` 21 |
| Outer / inner 50% / 80% | `outer_wall_speed` 30, `inner_wall_speed` 48 |
| Travel 6000 mm/min → 100 mm/s | `travel_speed` 100 |
| Quality Fast 0.30 / Medium 0.20 / High 0.10 | `0.30mm Extra Draft` / `0.20mm Standard` / `0.10mm Fine` |
| Thumbnail Qidi 300×300 | `thumbnails`: `300x300/M4010` |
| Dual-mode ooze shield + PVA on T1 | `support_filament` / `support_interface_filament` = 2 |
| Starting script (heat only) | `machine_start_gcode` (S3D-aligned; skirt primes, no startup wipe) |
| Chamber heat ABS/PA | `chamber_temperature` + `M191` wait when `overall_chamber_temperature > 0` |
| Pre tool-change X330/X0 | `toolchange_gcode` |

Built per [OrcaSlicer — How to create profiles](https://www.orcaslicer.com/developer_reference/how_to_create_profiles.html).

### Phase 2 — Dual extruder tuning

- [ ] Tool-change park positions (X0 / X330) per S3D/Qidi-Print
- [ ] Ooze shield / prime pillar / wipe tower strategy for PVA supports
- [ ] `switch_extruder_retraction` calibration starting from Qidi base (10 mm)
- [ ] Verify `M141`/`M191` chamber commands on real hardware (PLA → `M141 S0`; ABS → `M191 S55`)

### Phase 3 — G-code extras

- [ ] Enable QIDI thumbnails in printer preset if firmware supports them
- [ ] Post-process `M106 S0` → `M107` if needed (S3D did this)
- [ ] Validate prime line G-code (`G92 E-19` style) on real hardware

### Phase 4 — Network (optional)

- [ ] Evaluate CBD UDP agent vs. recommending USB/SD workflow
- [ ] Port relevant logic from `CBDConnect.py` if pursuing WiFi send

### Phase 5 — Upstream PR

- [ ] Test on physical I-Fast
- [ ] Document verification steps
- [ ] Open PR to [OrcaSlicer/OrcaSlicer](https://github.com/OrcaSlicer/OrcaSlicer) referencing #1784

---

## 10. Known gaps & open questions

| # | Question | Notes |
|---|---|---|
| 1 | Does I-Fast ship with Klipper or only CBD Marlin? | Qidi-Print says `board: CBD`; Orca Qidi agent is Moonraker-based |
| 2 | Exact extruder offsets (X/Y)? | S3D shows 0/0 — confirm on hardware |
| 3 | Which nozzle sizes are officially supported? | S3D shows 0.4; check QIDI docs / spare nozzles |
| 4 | Chamber temp G-code: `M141` vs other? | S3D uses `M141 S[chamber0_temperature]` |
| 5 | Is `G92 E-19` prime method required? | Present in Qidi-Print start G-code, absent in S3D default |
| 6 | Bed model / texture assets | Qidi-Print has `i-fast.stl`; need to license/redistribute or recreate |
| 7 | i-fast extruder definition files in Qidi-Print | Referenced but paths may differ — locate in Qidi-Print repo |
| 8 | Maximum chamber temperature in firmware? | Product says 60 °C; Qidi base allows up to 80 °C in slicer |
| 9 | Does firmware require QIDI-compressed G-code for WiFi? | `CBDConnect.py` supports `.gcode.tz` compression |

---

## 11. External links (quick index)

| Resource | URL |
|---|---|
| OrcaSlicer I-Fast issue | https://github.com/OrcaSlicer/OrcaSlicer/issues/1784 |
| Qidi-Print I-Fast definition | https://github.com/QIDITECH/Qidi-Print/blob/main/resources/definitions/i-fast.def.json |
| Qidi-Print CBD WiFi module | https://github.com/QIDITECH/Qidi-Print/blob/main/python/qidi/Wifi/CBDConnect.py |
| Qidi-Print repo | https://github.com/QIDITECH/Qidi-Print |
| I-Fast product page (EU) | https://eu.qidi3d.com/products/qidi-i-fast-a-pioneer-in-solving-complex-printing |
| I-Fast wiki | https://wiki.qidi3d.com/en/I-Fast |
| OrcaSlicer wiki | https://www.orcaslicer.com/wiki/ |
| Orca — how to create profiles | https://www.orcaslicer.com/developer_reference/how_to_create_profiles.html |

---

## 12. Discovery log

Append new findings here as the port progresses.

| Date | Finding |
|---|---|
| 2026-06-08 | Branch `QIDI_I-Fast_Port` created; S3D reference files added |
| 2026-06-08 | Initial knowledge base created from Qidi-Print, S3D, CBDConnect, Orca wiki, and product docs |
| 2026-06-08 | Orca `Qidi.json` has no I-Fast entry; `#1784` closed stale in upstream |
| 2026-06-08 | I-Fast uses CBD UDP:3000 — separate from Orca's Moonraker `QidiPrinterAgent` |
| 2026-06-08 | S3D and Orca both support QIDI thumbnail format (`thumbnail_QIDI`) |
| 2026-06-08 | Phase 1 Orca profiles added under `resources/profiles/Qidi/`; `orca_extra_profile_check.py --vendor=Qidi` passes |
| 2026-06-08 | Full S3D FFF comparison; presets rewritten to inherit `fdm_process_common` (not generic Qidi); see `QIDI_I-Fast_S3D_Files/S3D_ORCA_COMPARISON.md` |

---

## 13. File map (this port)

```
QIDI_I-FAST_PORT.md                          ← this document
QIDI_I-Fast_S3D_Files/
  Qidi Technology i-fast Printer Profile.fff ← S3D machine/process reference
  Qidi I-Fast Factory File.factory           ← S3D factory bundle
  readme.md
resources/profiles/Qidi/
  machine/Qidi I-Fast.json
  machine/Qidi I-Fast 0.4 nozzle.json
  process/fdm_process_qidi_ifast_common.json
  process/0.10mm Fine @Qidi I-Fast.json … 0.30mm Extra Draft @Qidi I-Fast.json
  Qidi I-Fast_cover.png                      ← placeholder art
src/slic3r/Utils/QidiPrinterAgent.*          ← newer QIDI network agent (Klipper)
src/libslic3r/GCode/Thumbnails.cpp           ← QIDI thumbnail encoder
```
