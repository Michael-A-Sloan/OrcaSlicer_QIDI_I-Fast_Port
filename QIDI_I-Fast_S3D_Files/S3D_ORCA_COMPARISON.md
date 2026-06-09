# S3D FFF → OrcaSlicer Profile Comparison

Source: `Qidi Technology i-fast Printer Profile.fff` (S3D 5.1.2, 2023-08-17)  
Target: `resources/profiles/Qidi/` I-Fast presets (2026-06-08)

S3D uses **mm/min** for speeds; Orca uses **mm/s** unless noted.

## Process presets (S3D Quality tiers)

| S3D preset | Layer | Top/Bottom | Skirt layers | Infill | Support infill | Orca preset |
|---|---:|---:|---:|---:|---:|---|
| Fast | 0.30 mm | 3 / 3 | 1 | 15% | 25% | `0.30mm Fast @Qidi I-Fast` |
| Medium (default) | 0.20 mm | 3 / 3 | 1 | 20% | 30% | `0.20mm Medium @Qidi I-Fast` |
| High | 0.10 mm | 4 / 4 | 2 | 30% | 40% | `0.10mm High @Qidi I-Fast` |

**Removed** (were generic Orca-style extras, not in S3D): `0.16mm Optimal`, `0.25mm Draft`, `0.20mm Standard`, `0.10mm Fine`, `0.30mm Extra Draft`.

## Process settings — S3D vs previous Orca vs now

| S3D FFF field | S3D value | Was (inherited Qidi common) | Now (I-Fast common) |
|---|---|---|---|
| outlinePerimeters | 2 | wall_loops **3** | wall_loops **2** |
| defaultPrintSpeed | 3600 mm/min (60 mm/s) | outer 25, travel 150 | outer **30**, travel **100** |
| outerPerimeterSpeed % | 50% | 25 mm/s | **30** mm/s |
| innerPerimeterSpeed % | 80% | 40 mm/s | **48** mm/s |
| firstLayerSpeed % | 35% | 15 mm/s | **21** mm/s |
| firstLayerWidth % | 125% | 0.5 mm | **0.5** mm (kept) |
| sparseInfillPattern | rectilinear | crosshatch | **grid** |
| sparseInfill % (medium) | 20% | 15% | **20%** |
| infillOutlineOverlap % | 15% | 23% | **15%** |
| skirtOffset / outlines | 4 mm / 2 | 2 mm / 1 | **4** mm / **2** |
| accelXY / Z / E | 1000 / 150 / 3000 | 500 / 100 / 5000 | **1000** / Z via machine / E via machine |
| jerkXY | 600 mm/min (10 mm/s) | 8 | **10** |
| sparseSupport % | 30% | — | **30%** (medium) |
| supportHorizontalPartOffset | 0.3 mm | 0.35 | **0.3** |
| useOozeShield (dual mode) | 1 | — | ooze_prevention **1** |
| usePrimePillar (dual mode) | 0 | enable_prime_tower **1** | enable_prime_tower **0** |
| support on left extruder | T1 | support_filament 2 | support_filament **2** |

## Machine settings

| S3D FFF field | S3D value | Was | Now |
|---|---|---|---|
| buildVolume | 330×250×320 | ✓ | ✓ |
| retractDistance | 1.5 mm | ✓ | ✓ |
| retractSpeed | 1800 mm/min (30 mm/s) | ✓ | ✓ |
| toolChangeRetractDistance | 12 mm | ✓ | ✓ |
| toolChangeExtraRestartDistance | -0.5 mm | 0 (Qidi common) | **-0.5** |
| toolChangeRetractSpeed | 600 mm/min (10 mm/s) | — | uses retraction_speed 30* |
| retractVerticalLift | 0 | z_hop 0.4 (Qidi common) | z_hop **0** |
| useWiping | 0 | wipe 1 | wipe **0** |
| speedMaxFlowRate | 900 mm³/min | — | filament max_vol **15** mm³/s |
| thumbnail qidi 300×300 | yes | ✓ COLPIC | ✓ |
| startingScript (dual) | heat + M141 | Qidi-Print prime only | S3D heat + **M141** + prime |
| endingScript | Z320, X360 Y250 park | partial | **full S3D park** |
| preToolChange X330/X0 | yes | ✓ | ✓ |

\* Orca has no separate tool-change retract speed; S3D 10 mm/s vs normal 30 mm/s — note for hardware tuning.

## Filament presets (S3D Material auto-configure)

| S3D material | Nozzle | Bed | Chamber | Fan L1 / L2+ | Orca preset |
|---|---:|---:|---:|---|---|
| PLA | 210 | 60 | 0 | 0% / 100% | `Generic PLA @Qidi I-Fast` |
| ABS | 230 | 100 | 0 (S3D) / **55** (Orca heated) | 0% | `Generic ABS @Qidi I-Fast` |
| PETG | 235 | 75 | 0 | 0% | `Generic PETG @Qidi I-Fast` |
| PVA | 195 | 80 | 0 | 0% / 100% | `Generic PVA @Qidi I-Fast` |
| Nylon | 225 | 80 | 0 | 0% | `Generic PA @Qidi I-Fast` |

Previously only generic `Qidi Generic PLA` (Klipper-era temps/PA) was referenced — not S3D-tuned.

## S3D settings with no direct Orca equivalent

| S3D | Notes |
|---|---|
| coasting / wiping distances | Orca uses different combing/wipe model |
| reduceSpeedForQuickLayers | Use `slow_down_layer_time` on filaments instead |
| steps/mm X/Y/Z/E | Firmware-side; not in Orca profiles |
| postProcessing M106→M107 | May need post-process script if required on hardware |
| Prime pillar (single-tool S3D) | Disabled in S3D dual mode; Orca uses ooze shield |

## Factory file

`Qidi I-Fast Factory File.factory` is a binary S3D bundle. The `.fff` XML above is the authoritative extracted reference; factory file was not separately parsed.
