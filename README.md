# Antenna Calculator PRO — Version 1.19.264

**Android application for antenna analysis, MMANA models, electromagnetic-field visualization, reflector/feed studies, and LiteVNA64 measurements**  
**© UA1CFM, 2026** · **English** · [Русская версия](README_RU.md)

> **Source of truth:** `Antenna_Calculator_PRO_1.19.264_SELECTABLE_CU_WIRE_LOSS_TOP_FOLDER(1).zip`. This README describes capabilities grounded in that source tree. Older audit reports shipped in the archive are historical evidence, not proof that all features of the current build have been independently validated.

## Contents

1. [Overview](#overview)
2. [What's new in 1.19.264](#whats-new-in-119264)
3. [Features](#features)
4. [Numerical solver and accuracy](#numerical-solver-and-accuracy)
5. [Quick start](#quick-start)
6. [MAA models and antenna stacks](#maa-models-and-antenna-stacks)
7. [Near Field and currents](#near-field-and-currents)
8. [Feed and reflector tools](#feed-and-reflector-tools)
9. [LiteVNA64 and Touchstone](#litevna64-and-touchstone)
10. [Building the Android project](#building-the-android-project)
11. [Validation and limitations](#validation-and-limitations)
12. [Reporting issues and credits](#reporting-issues-and-credits)

## Overview

Antenna Calculator PRO brings together MMANA `.maa` wire-antenna analysis, 2D/3D far-field pattern visualization, near-field estimates, parabolic-reflector/feed utilities, CST `.ffs` and HFSS `.ffd` parsing, Touchstone data, and LiteVNA64 USB measurement tools. It is intended for amateur-radio research, design exploration, and engineering diagnostics, not certified metrology.

The supplied project is implemented in **Kotlin / Jetpack Compose** and includes antenna assets, numerical modules, diagnostic records, and Android build files.

## What's new in 1.19.264

- **WIRE MATERIAL** selection in MAA analysis: ideal **PEC** or **copper (Cu)** with conductivity **5.8 × 10⁷ S/m**.
- Optional ohmic wire-loss treatment; batch CSV records the selected material.
- Preserves **1.19.263** fixes for Russian MMANA combined/telescopic-wire section `Установки комб. пров.` and its `L0/L1/...` length semantics.
- **Known limitation:** impedance remains diagnostically wrong for some DDRR and GP+LC designs; copper-loss support does **not** fix these cases.
- The included `WHATS_NEW_1.19.264_CU_LOSS_RU.txt` reports standalone JVM checks of the Kotlin core, **not** a completed Android Gradle build of the delivered archive.

## Features

| Area | Functions |
|---|---|
| MMANA / MAA | Antenna-file/library loading, wires, source and load definitions, selected stack configurations, current and impedance solving |
| Radiation patterns | 2D cuts, interactive 3D views, polarization tools |
| Frequency analysis | Complex input impedance, SWR, Smith chart and sweep-oriented views |
| Near Field | E/H components, B/S-related representations, planar slices and phase animation |
| Feeds and reflectors | CST `.ffs` / HFSS `.ffd`, reflector illumination, phase and phase-center diagnostics |
| RF tools | Calculators and reference utilities in `HamTools` |
| Touchstone | `.s1p` / `.s2p`, S-parameter traces and markers |
| LiteVNA64 | USB interface, live frequency sweep, calibration and data analysis |
| Audio | Experimental RF sonification and HRTF-based presentation |
| Batch diagnostics | CSV output, batch processing and resume-related code |

> A source-code module's presence does not guarantee that every UI action works identically across all Android phones or versions.

## Numerical solver and accuracy

**The primary MAA calculation engine in v1.19.264 is an in-project MININEC-like solver, not NEC-2 FULL.** `MmanaSolverApi.solveAdaptive()` delegates to `MmanaMininecSolver.solve()`. The shipped `SOLVER_AUDIT_1.19.250.txt` confirms this architecture; `Nec2Core` and `Nec2FullDeck` remain for export/shared structures and should not be cited as proof that NEC-2 FULL drives MAA impedance.

Numerical correctness depends on wire geometry and radii, segmentation, source/load positions, ground conditions, and convergence. Complex impedance, currents, and radiation patterns warrant comparison with MMANA, CST, or measured references, particularly for DDRR, LC matching networks, coupled multi-source arrays, and lossy ground.

**Near-field E/H plots have a physical computation basis but are not independently certified quantitative field results.** Use them first for patterns and relative behavior; verify absolute values with well-controlled references.

## Quick start

1. Install an APK you built yourself or obtained from a trusted project release.
2. Open **ANTENNA / MAA** and choose a bundled model or import an external `.maa` file.
3. Check frequency, geometry, sources, loads, and ground setting.
4. Select **WIRE MATERIAL** (`PEC` or `Cu`) according to the intended study.
5. Run the calculation and review R+jX, SWR/Smith, 2D/3D patterns, currents, and Near Field.
6. For unexpected results, verify the model against an independent solver rather than adjusting values to match expectations.

## MAA models and antenna stacks

The project includes MMANA assets, library indexes, and `maa_library` content. It addresses wire antennas such as dipoles, Yagis, verticals, loops, and selected Make Stack designs. Correct stack analysis requires expanding every physical antenna and preserving source count, coherent phases/amplitudes, load placement, and mutual coupling.

In MMANA files, a negative wire-radius field may reference a **combined/telescopic wire** rather than denote a physically negative radius. Version 1.19.263 adds recognition of the Russian combined-wire section; especially demanding geometries still merit independent inspection.

## Near Field and currents

`NearFieldEngine.kt` evaluates electromagnetic quantities from complex segment currents; `NearFieldSliceGeometry.kt` provides spatial slice geometry. Displays include E/H and derived B/S quantities as well as phase-dependent visualization. A minimum-distance regularization is applied near current elements, making very-near-wire and feed-junction values less suitable as quantitative references.

When comparing against CST, match antenna geometry, power/current normalization, frequency, coordinate axes, phase convention, and observation points. Compare complex **Ex/Ey/Ez** and **Hx/Hy/Hz**, not only plot colors or normalized maxima.

## Feed and reflector tools

`FfsParser`, `HfssFfdParser`, `FeedDiagnostics`, `PhaseAnalysis`, `AntennaMath`, and related screens support imported feed far-field data and parabolic-reflector analysis. Check input frequency, angular sampling, polarization, phase convention, and orientation carefully. Analytical feed/reflector estimates do not replace a full-wave CST reflector simulation.

## LiteVNA64 and Touchstone

The source includes `LiteVnaUsb`, `LiteVnaProtocol`, `LiteVnaCalibration`, `LiteVnaLiveController`, `Touchstone`, and `TouchstoneTab`. These tools handle measurement traces, markers, calibration workflows, and S-parameter exchange. Measurement reliability depends on the device, calibration, cabling, frequency span, and USB link. VNA measurements are separate from simulated MAA values.

## Building the Android project

The project config declares **compileSdk 35**, **minSdk 24**, **targetSdk 35**, Java/Kotlin **17**, Kotlin and **Jetpack Compose**, Compose BOM `2024.12.01`, AndroidX Activity Compose `1.10.1`, and coroutines `1.9.0`. Install Android Studio, the necessary Android SDK and compatible JDK, then run:

```bash
./gradlew assembleDebug
```

On Windows:

```bat
gradlew.bat assembleDebug
```

A successful debug build typically outputs `app/build/outputs/apk/debug/`. **The supplied changelog does not establish a successful full Android Gradle build for v1.19.264**; resolve build problems using your local SDK/Gradle diagnostics.

## Validation and limitations

- The archive includes `SOLVER_AUDIT_1.19.250.txt`, prior `VALIDATION_*`, `AUDIT_*`, `SUMMARY_*` reports, and test cases. These document earlier tests; they are **not** certification for every feature of v1.19.264.
- According to `WHATS_NEW_1.19.264_CU_LOSS_RU.txt`, some DDRR and GP+LC impedance results are still wrong.
- Absolute near-field E/H near wires, sources, and load junctions, plus complicated phased stacks, need independent validation.
- Ground models, conductivity, thin-wire approximations, and matching networks can introduce material numerical differences.
- Near-field color maps and sonification are exploratory representations, **not calibrated measurement instruments**.
- The archive includes third-party antenna data and HRTF/KEMAR assets. Review redistribution permissions and provide relevant notices before publishing the complete source bundle. A project-level open-source `LICENSE` should be chosen separately; do not assume bundled third-party files share the same terms.

## Reporting issues and credits

For a reproducible bug report, provide the source `.maa` / `.ffs` / `.ffd` / `.s1p`, frequency, ground setting, wire material, screenshots, expected impedance/pattern/field results, and independent MMANA/CST or measurement evidence.

**Antenna Calculator PRO · © UA1CFM, 2026.**
