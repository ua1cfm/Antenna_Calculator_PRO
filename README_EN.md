# Antenna Calculator PRO for Android

**Version 1.19.269 · UA1CFM · 2026**  
**English** | [Русский](README_RU.md)

**Antenna Calculator PRO** is an Android application for radio amateurs and RF engineers combining MMANA-GAL wire-antenna model analysis, CST/HFSS far-field data processing, and RF measurement utilities. It provides complex input impedance, radiation patterns, current distribution, near-field visualization, and additional antenna and measurement tools.

> **Project status:** actively developed engineering software. Numerical results should be checked against reference simulations and measurements; accuracy varies by antenna topology, load, and ground model.

## Features

### MMANA-GAL antenna models (`.maa`)
- Import wire-antenna geometry, excitations, loading elements, and ground settings.
- Calculate **Z = R + jX**, SWR, and related feed-point parameters.
- Kotlin-based **MMANA-compatible MININEC engine** for currents, impedance, and radiation-pattern input data.
- Multiple coherent sources, special `W#E` pulse addressing, combined/telescopic wires (CombWire), and Make Stack models.
- Analyze Yagi antennas, arrays, verticals, loops, and other wire geometries, subject to independent validation of difficult cases.
- Optional copper conductor-loss modeling versus PEC conductors.
- Batch processing, diagnostics, and CSV export.

### Radiation patterns and 3D visualization
- 2D patterns, principal-plane analysis, and directivity-related measurements.
- Interactive 3D patterns overlaid on the antenna geometry, including wires, source markers, and loads.
- Translucent 3D surface shading with near/far face differentiation during rotation.
- PNG image export and interactive geometry viewing.
- Current-distribution, phase, and phase-animation capabilities.

### Fields, feed analysis, and simulation imports
- Near-field visualization including E/H quantities and derived field views where supported by the active mode.
- Import **CST `.ffs`** and **Ansys HFSS `.ffd`** far-field data.
- Radiation, phase, feed, and reflector analysis from simulation results.
- Smith Chart, frequency sweeps, and impedance-matching tools.

### Measurements and utilities
- **LiteVNA64** USB connectivity on supported Android USB Host/OTG devices.
- S-parameters, S11/S21, SWR, markers, sweeps, and Touchstone files.
- Measurement and utility features including TDR and experimental RF data sonification.
- General amateur-radio calculation utilities and unit conversions.

## Solver architecture

The current primary solver for **MAA feed-point impedance is the MMANA/MININEC implementation**, **not the former NEC-2 FULL engine**. Some `Nec2*`-named source files are still retained for auxiliary/compatibility functionality; their presence does not mean they are the active default impedance solver.

Numerical parity with MMANA-GAL is **not guaranteed** for every design. Finite-conductivity ground, LC loads, connected-wire junctions, segmentation, and multiport models require careful regression testing. A completed calculation should not be interpreted as proof of physical accuracy.

## What's new in 1.19.269

- MININEC endpoint connection tolerance raised to at least **10 µm (1×10⁻⁵ m)**, aligned with Windows v0.4.8 work.
- Improved near/far translucency and shading in the 3D radiation view, including PNG export.
- Existing `W#E` pulse handling, source orientation, and multi-source support are preserved rather than rewritten without tests.
- **Not ported:** the Windows C# `MmanaReferenceEngine` as a replacement for the Kotlin solver, or the Windows glass rasterizer verbatim.

## Requirements

- **Android 7.0 (API 24)** or later; target SDK 35.
- USB Host/OTG support and a compatible device for LiteVNA64 connectivity.
- To build from source: **Android Studio**, Android SDK 35, and compatible JDK (the Gradle project targets Java/Kotlin 17).

## Building from source

1. Download and extract the complete source package.
2. Open its root folder in Android Studio (`settings.gradle.kts`, `build.gradle.kts`, `app/`, and `gradlew.bat` are located there).
3. Wait for Gradle synchronization and install SDK 35 if prompted.
4. Select `app` and run **Build → Build APK(s)**, or use `gradlew.bat assembleDebug` on Windows.
5. Install the APK on an Android device and approve USB access when connecting external instruments.

Package identifier: `ua1cfm.cstffs`. Version: **1.19.269** (`versionCode 119269`).

## Validation and known limitations

- Source changes for 1.19.269 were inspected, but a complete Android Gradle build and on-device test **have not been verified** for this release.
- Full numerical equivalence between Windows `MmanaReferenceEngine` and the Kotlin MININEC solver has not yet been established.
- Complex designs involving real ground, LC loads, or unusual segmentation may disagree significantly with MMANA-GAL.
- Recommended regression cases include **Coil vertical**, loaded Yagi designs, Make Stack models, and multiple-source antennas.
- Instrument measurements require appropriate calibration and independent verification.

## Repository layout

```text
app/
  src/main/java/ua1cfm/cstffs/
    MainActivity.kt
    MmanaMininecSolver.kt
    MmanaSolverApi.kt
    MaaParser.kt
    Radiation3D.kt
    ThreeDTab.kt
    NearFieldEngine.kt
    FfsParser.kt
    HfssFfdParser.kt
    LiteVnaUsb.kt
    LiteVnaProtocol.kt
    ...
build.gradle.kts
settings.gradle.kts
gradlew.bat
```

## Bug reports and feedback

Please attach a reproducible `.maa`/`.ffs`/`.ffd` input file when redistribution is permitted, the app version and frequency, ground settings, calculated `R+jX`, expected MMANA-GAL values, and screenshots showing the discrepancy. For LiteVNA64 issues, include device/firmware and calibration details.

## Credits

**UA1CFM · 2026**. Developed for amateur-radio experimentation and engineering analysis. Licensing terms should be established separately by the repository owner; publishing source code on GitHub does not automatically grant an open-source license.
