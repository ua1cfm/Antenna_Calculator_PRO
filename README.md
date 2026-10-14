# Antenna Calculator PRO — v1.19.267

**Android antenna modelling, RF measurement and reflector/feed analysis**  
**© UA1CFM, 2026** · [Русская документация](README_RU.md)

> **Release status:** Experimental source release. The v1.19.267 sweep fallback guard has undergone source-level inspection, but this package has **not** passed a complete fresh Android build, all-model numerical regression, or new authoritative MMANA comparison. Previously collected validation files are historical evidence, not results of v1.19.267.

## Contents

1. [Purpose and scope](#purpose-and-scope)
2. [Verified project metadata](#verified-project-metadata)
3. [Features](#features)
4. [Numerical engine](#numerical-engine)
5. [MMANA input and modelling assumptions](#mmana-input-and-modelling-assumptions)
6. [Impedance and frequency-domain sweeps](#impedance-and-frequency-domain-sweeps)
7. [Electric and magnetic fields](#electric-and-magnetic-fields)
8. [Radiation patterns and reflector analysis](#radiation-patterns-and-reflector-analysis)
9. [LiteVNA, Touchstone and utilities](#litevna-touchstone-and-utilities)
10. [Installing and building](#installing-and-building)
11. [Architecture and source map](#architecture-and-source-map)
12. [Validation and limitations](#validation-and-limitations)
13. [Troubleshooting](#troubleshooting)
14. [Reproducible bug reports](#reproducible-bug-reports)
15. [Contributing, licensing and acknowledgments](#contributing-licensing-and-acknowledgments)

## Purpose and scope

Antenna Calculator PRO combines thin-wire antenna analysis, import of electromagnetic far-field data, interactive 2D/3D radiation visualization, feed/reflector estimates, near-field visualization, impedance plots, measurement workflows and amateur-radio engineering tools in an Android app.

The software is intended for engineering exploration and comparative modelling. It is **not** a certified measurement tool, full 3D FEM/FDTD solver, or a drop-in replacement for every MMANA-GAL numerical convention. Model validity depends on topology, frequency, excitation, segmentation, conductor materials, loads and ground.

## Verified project metadata

| Item | Value in this source release |
|---|---|
| Application ID / Kotlin package | `ua1cfm.cstffs` |
| Version name | `1.19.267` |
| Version code | `119267` |
| Build system | Android Gradle Plugin / Gradle Kotlin DSL |
| UI technology | Kotlin and Jetpack Compose / Material 3 |
| `compileSdk` / `targetSdk` | 35 / 35 |
| `minSdk` | 24 |
| Java/Kotlin JVM target | 17 |
| Source license | **No project-level license should be assumed**; see licensing section |

The `gradle/wrapper/gradle-wrapper.jar` file is absent from the source archive inspected for this documentation. Consequently, a clean `./gradlew` build cannot be promised without repairing the wrapper or using a compatible local Gradle installation. Install a suitable Android SDK (API 35), JDK 17 or compatible toolchain, and Android Studio.

## Features

### MMANA `.maa` antenna projects

- Read wire-based antenna geometries and associated model data through `MaaParser.kt`.
- Display wire geometry, inspect impedance and render computed radiation patterns.
- Support multiple excitations, matching networks and stack configurations in model-dependent paths; complex cases require additional validation.
- Evaluate conductor-loss options, including PEC and copper (`Cu`) selection in the 1.19.264 lineage.
- Provide Smith-chart impedance trajectories and an interactive frequency sweep.
- Preserve physically computed R+jX instead of replacing it with a preferred 50/75/100 Ω result.

### Field and RF analysis

- 2D principal-plane and 3D radiation-pattern display.
- Current magnitude, current phase and time-phase animation based on solved complex currents.
- Near-field electric and magnetic components, magnetic-flux density and power-flow-related displays.
- Far-field import and analysis for CST `.ffs` and HFSS `.ffd` formats.
- Parabolic-reflector / feed calculations and supporting RF tools.
- LiteVNA64, Touchstone `.s1p`/`.s2p`, Smith/SWR/return loss and related sweep utilities.
- RF audio/sonification, tools for antenna design, and batch-oriented components whose availability depends on the current UI and connected hardware.

**Implementation note:** A source module being present establishes the existence of that implementation area; it does not alone guarantee the function works on every Android device or that all historical UI controls remain exposed in this exact revision.

## Numerical engine

The production MMANA impedance path is:

```text
MAA geometry / material / source / ground
           |
       MaaParser
           |
  MmanaSolverApi.solveAdaptive(...)
           |
  MmanaMininecSolver.solve(...)
           |
  complex segment currents, input impedance, radiation data
```

This branch uses an in-project **MININEC-compatible wire solver** in `MmanaMininecSolver.kt` as the principal MAA numerical engine. It should **not** be advertised as computing every MAA case with `NEC-2 FULL` merely because legacy NEC-named files remain in the source tree. `Nec2FullDeck.kt` and `Nec2Core.kt` also serve export/shared-data needs.

Numerical fidelity is sensitive to pulse topology, junction orientation, matrix conditioning, geometry scale, ground conditions, segmentation and placement of sources/loads. There is no permission to replace unconverged answers with standard impedance values or silently manufacture smooth responses.

## MMANA input and modelling assumptions

`.maa` files can include named wire coordinates, radii, segmentation tags, sources, loads, earth settings, and compact multi-antenna/stack metadata. These must be treated as **electrical** as well as **geometric** data. Small mistakes in endpoint pulses, stacked excitations or matching networks can cause large errors in R+jX.

Special cases requiring care include automatically segmented short wires, stepped/tapered elements, DDRR and fractal structures, complex loaded junctions, phased arrays, real-ground models and multiple coherent sources. The program's model-processing utilities include `MmanaGalSegmentation.kt`, `MmanaStackSegmentation.kt`, `MmanaMultiSourceReference.kt`, and ground-model modules. Do not assume that visual agreement of 3D geometry proves identical numerical discretization.

## Impedance and frequency-domain sweeps

The impedance is complex, `Z(f) = R(f) + jX(f)`, and displayed relative to a chosen Smith reference impedance `Z0`. **`Z0` is a visualization/SWR reference**; the engine does not tune the antenna's physical impedance to that value.

### What changed in the recent versions

| Version | Source-level change | What it does **not** prove |
|---|---|---|
| 1.19.264 | Copper/PEC wire-loss selection in the supplied lineage | Absolute agreement of all loaded/resonant MAA models |
| 1.19.265 | Experimental Tei-based segmentation for a restricted dense-short-wire population with `EC ≤ 1` | A universal equivalent of MMANA segmentation |
| 1.19.266 | Freeze pulse discretization at the sweep reference frequency and calculate frequency-dependent impedance using that common topology; 61 frequency samples with greater center density | That each impedance value or resonance frequency matches MMANA |
| **1.19.267** | Guard against switching to a multi-source real-ground fallback which has a different discretization during a frozen-topology sweep | A fix for all negative-R cases or a successful full-corpus regression |

In 1.19.267, if `activeSources >= 2`, the model uses real ground, and the primary result yields negative R, the fallback path could otherwise introduce a point computed using another mesh. With a sweep reference supplied, the application instead **stops the sweep with an explicit diagnostic**. Ordinary single-frequency fallback behavior remains unchanged. This is a continuity/safety guard, not impedance curve smoothing.

The user-observed improvement for DDRR-DL2KQ in 1.19.266 was a **smooth Smith trace**, after visibly discontinuous earlier traces. One screenshot validates the displayed behavior for that example, not numerical error bounds or all-model performance.

### How to verify a sweep

Compare each point's actual `R(f)` and `X(f)`, segment count, excitation pulse, matrix residual and solver status. Use a dense local frequency interval for narrowband resonators, rather than inferring resonance from a coarse ±5% trace. Check that all points have the same pulse topology while retaining the **actual** frequency in Maxwell's equations.

## Electric and magnetic fields

`NearFieldEngine.kt` reconstructs fields from complex segmented currents. The near-field section provides components such as E and H, plus B/power-flow-related quantities and plane slices. The physical model includes distance-dependent reactive, induction and radiation terms. It is **not** a general 3D volumetric FEM/FDTD field solver.

The near-wire evaluation regularizes the minimum observation distance to avoid singular or unstable values; current interpolation and segment approximations can alter the field close to metal surfaces, feed nodes, traps and lumped networks.

**Practical interpretation:** relative distributions and field topology are useful for exploration; absolute V/m, A/m, local phase and fields extremely close to wires remain unvalidated for universal precision. For independent checks, compare complex `Ex/Ey/Ez` and `Hx/Hy/Hz` at identical observation points with a trusted solver, with matching excitation and ground assumptions. In the far field, the magnitude of the free-space E/H ratio tends toward approximately 377 Ω; this need not hold in the reactive near field.

## Radiation patterns and reflector analysis

The project contains `MmanaRadiationPattern.kt`, `Radiation3D.kt`, `ThreeDTab.kt`, `FfsParser.kt`, `HfssFfdParser.kt`, `FeedDiagnostics.kt`, and reflector/antenna mathematics. Available presentations include 2D cuts, interactive 3D plots and feed diagnostics. Far-field files imported from CST/HFSS are externally computed field data and should be distinguished from patterns generated internally from MAA currents.

For reflector work, align angle conventions, polarization, phase center, frequency, feed placement and normalization. Spillover/taper/efficiency estimates are only as reliable as their inputs and the model used.

## LiteVNA, Touchstone and utilities

Relevant source modules include `LiteVnaUsb.kt`, `LiteVnaProtocol.kt`, `LiteVnaCalibration.kt`, `LiteVnaLiveController.kt`, `Touchstone.kt`, `TouchstoneTab.kt`, `TdrMath.kt`, `RfSonification.kt`, `AntennaSonification.kt`, `HamTools.kt` and `YagiDesigner.kt`.

The app includes or has source implementations for USB VNA acquisition, calibration-related logic, S-parameter traces, marker-style analyses, Touchstone I/O, RF calculations, antenna-design assistance and sonic representation of RF curves. Hardware support is contingent on Android USB permissions, compatible VNA firmware, sampling settings and correct calibration. Do not interpret simulated impedance as measured S11 without a calibrated acquisition chain.

## Installing and building

1. Extract the full **Antenna Calculator PRO 1.19.267** source archive to a writable folder. Open the directory that directly contains `settings.gradle.kts` and `app/` in Android Studio.
2. Install Android SDK API 35 and a suitable JDK 17-based toolchain. Allow Gradle to synchronize dependencies using network access.
3. **Check the Gradle wrapper:** the source package inspected here has no `gradle/wrapper/gradle-wrapper.jar`. Recover a legitimate wrapper with a compatible Gradle version using a local Gradle installation, or use the IDE's supported tooling. Do not download an arbitrary unverified JAR.
4. Select an Android emulator or a compatible device (Android API 24+), and build/run the `app` module.
5. Import a small known MAA model first, then check geometry, feed and `R+jX` before drawing conclusions from fields/3D plots.

No clean Android APK build of this particular release was verified in the current source audit. The presence of source code and Gradle declarations does not constitute a tested binary release.

## Architecture and source map

All paths below are relative to the Android project root:

| Area | Files |
|---|---|
| UI and navigation | `app/src/main/java/ua1cfm/cstffs/MainActivity.kt`, `YagiMaaTab.kt`, `ThreeDTab.kt` |
| Input/parser | `MaaParser.kt`, `FfsParser.kt`, `HfssFfdParser.kt`, `Touchstone.kt` |
| Electromagnetics | `MmanaSolverApi.kt`, `MmanaMininecSolver.kt`, `MmanaRadiationPattern.kt` |
| Segmentation / stacks / ground | `MmanaGalSegmentation.kt`, `MmanaStackSegmentation.kt`, `MmanaMultiSourceReference.kt`, `GroundModelV2.kt` |
| Near fields and polarization | `NearFieldEngine.kt`, `NearFieldSliceGeometry.kt`, `PolarizationMath.kt`, `PhaseAnalysis.kt` |
| 3D rendering | `Radiation3D.kt`, `SystemPattern3D.kt`, `ThreeDTab.kt` |
| VNA and data | `LiteVnaUsb.kt`, `LiteVnaProtocol.kt`, `LiteVnaCalibration.kt`, `LiteVnaLiveController.kt`, `TouchstoneTab.kt` |
| Other RF tools | `YagiDesigner.kt`, `HamTools.kt`, `FeedDiagnostics.kt`, `TdrMath.kt`, `RfSonification.kt` |
| Build | `app/build.gradle.kts`, `build.gradle.kts`, `settings.gradle.kts`, `gradle.properties` |

Historical `VALIDATION_*`, `AUDIT_*`, `WHATS_NEW_*` and `EXPERIMENT_*` files are included in the wider source archive. Always consult their recorded version before quoting them as evidence for v1.19.267.

## Validation and limitations

### Confirmed from source inspection

- Version identifiers and Android build configuration are consistent with **1.19.267**.
- The principal MAA engine is the project's MININEC-like solver, not automatically the legacy NEC-2 native path.
- v1.19.266's fixed-topology sweep mechanism remains; v1.19.267 adds a guard on the incompatible multi-source real-ground fallback.
- The modification does not by itself introduce smoothing, resonance fitting, or forced target impedances.

### Not confirmed for this release

- Full Android compilation and end-to-end operation on diverse phones.
- A fresh numerical sweep of the complete 1713–2435 MAA collection using the v1.19.267 binary.
- Quantitative MMANA comparison of every model and all exceptional source/load/ground combinations.
- Absolute accuracy of near-field E/H components or a complete 3D cross-solver validation.

Particularly sensitive antennas include DDRR and other high-Q resonators, fat/tapered elements, fractals, gamma/LC-matched systems, PEC/real-ground arrays and coherent multi-feed stacks. The special fallback guard **may terminate a sweep** rather than show results computed with inconsistent topologies. Treat that diagnostic as intentional, not as a successful electromagnetic solve.

## Troubleshooting

| Symptom | Check |
|---|---|
| `gradlew` cannot start | Ensure the Gradle wrapper JAR and scripts are present and compatible |
| Android Studio sync fails | JDK/Gradle/AGP alignment, SDK 35 and dependency repositories |
| Unrealistic R or X | Geometry units, pulse topology, wire radii, loads, source orientation, ground |
| Sharp Smith jumps | Are segments fixed across the sweep? Was a fallback invoked? Inspect diagnostic output |
| Narrow DDRR resonance appears shifted | Increase local frequency resolution; compare segment locations and MMANA reference |
| Near-wire E/H spikes or odd minima | Regularized observation distance and finite-segment model limitations |
| LiteVNA has unreliable traces | USB link, permissions, calibration, firmware, sample averaging and cables |

## Reproducible bug reports

Please include: app build/version, original `.maa`/`.ffs`/`.ffd`/`.s1p` source, center/sweep frequencies, coordinate units, source and load locations, PEC/real ground and soil assumptions, expected MMANA/CST values, exact error text, output traces or CSV, and device/Android version. For numerical discrepancies, provide both `R(f)` and `X(f)`, not just a cropped Smith-chart screenshot.

A separate **JVM real-solver testbench** was assembled for 1.19.267, but successful compilation and numerical execution were *not* established during its preparation. It must not be cited as an independently verified regression run.

## Contributing, licensing and acknowledgments

**Author:** UA1CFM · **Copyright:** © 2026 UA1CFM.

The availability of source code does **not** itself confer permission to redistribute or relicense third-party code, historical antenna corpora, sample models, HRTF assets or bundled vendor-related materials. Before public release, review all dependencies and bundled assets, preserve required attribution, add an explicit project license for the original code, and document rights to each third-party component. No project-level license grant is asserted here.

Contributions should preserve physical validity, keep numerical reference data separate from fitted assumptions, and report failures instead of inventing corrected impedance values.
