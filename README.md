WDDM-Hybrid-Graphics-Orchestrator

1. Overview

The WDDM-Hybrid-Graphics-Orchestrator is an elite-grade, privacy-first, 100% offline Windows application designed to manage Direct3D/DXGI cross-adapter rendering handoffs. It ensures stable, tear-free frame delivery from a discrete GPU (Pascal-architecture and newer) through an integrated GPU (or secondary display adapter) bypassing Windows Desktop Window Manager (DWM) desync issues. It features a hardware-accelerated, transparent, real-time Heads-Up Display (HUD) overlay for tracking performance metrics (FPS, Core Temperature, Clock Speed, and Memory Usage) with zero network overhead and zero third-party telemetry.

2. What the PowerShell Orchestration Script Does

The Build-GpuMonitor.ps1 script is a mathematically idempotent, zero-touch build pipeline. Executing this single script performs the entire lifecycle from raw source code to final packaged executable.

Specifically, the script executes the following phases:

Stale Process Termination: Sweeps memory for active background instances of the application or the C# bridge to release file locks.

Workspace Structuring: Generates the precise src/ package layout and directories (build/, dist/, logs/, bridge_src/) at the target path (F:\imonlinegaming\WDDM-Hybrid-Graphics-Orchestrator).

BOM-Free File Writing: Utilizes .NET System.Text.UTF8Encoding to write all source files without Byte-Order Marks (BOM), ensuring compiler and interpreter compatibility.

Native C# Compilation: Writes the raw C# DirectX Graphics Infrastructure (DXGI) code and invokes the built-in Windows .NET framework compiler (csc.exe) to build a native, headless binary (BridgeLayer.exe).

Python Architecture Generation: Constructs the complete Python application using strictly the Standard Library and raw ctypes for Win32 API interactions.

VENV Isolation: Creates a highly isolated Python virtual environment (.venv) to ensure no host-system package contamination.

PyInstaller Hardening: Invokes PyInstaller using a custom .spec manifest that dynamically strips out all unnecessary POSIX libraries, injects the native XML application manifest (for DPI awareness), and packages the Python scripts and the C# binary into a single, self-contained .exe.

3. What the Script Creates (Directory Architecture)

The orchestrator generates the following strict layout:

bridge_src/Bridge.cs: The raw C# code defining the DXGI Direct Flip swapchain.

src/utils/BridgeLayer.exe: The compiled C# sub-process binary.

src/utils/logger.py: A hierarchical, rotating file logger outputting to the logs/ directory.

src/utils/credentials.py: A pure ctypes implementation of the Win32 CREDENTIALW structure for secure, offline API/Credential Manager integration.

src/utils/hardware_metrics.py: A hardware telemetry poller utilizing subprocess calls to the native Windows nvidia-smi.exe and user32.EnumDisplaySettingsW for zero-overhead polling.

src/ui/overlay_manager.py: The completely decoupled, hardware-accelerated Tkinter overlay HUD. It handles click-and-drag logic, transparent background rendering via #000001 color-keying, and custom right-click context menus via Win32.

src/ui/app_window.py: The primary GUI instance that binds the logical queue to the UI, allowing GPU selection and Bridge initialization.

src/main.py: The standard entry point.

build_spec.spec: The hardened PyInstaller configuration file.

dist/WDDM-Hybrid-Graphics-Orchestrator.exe: The final, ship-ready executable.

4. Application Functionality & Features

Stable Cross-Adapter Handoff

By selecting a specific Pascal-or-newer GPU in the main GUI and clicking "Initialize DXGI Handoff Bridge", the Python application silently spawns BridgeLayer.exe. This C# binary hooks into dxgi.dll, creates an independent SwapChain with DXGI_SWAP_EFFECT_FLIP_DISCARD, and configures triple-buffering. This enforces direct hardware flipping to bypass DWM compositing lag.

Transparent Real-Time HUD Overlay

When initialized, the application detaches a 120x120 Heads-Up Display.

Always-On-Top: Aggressively forced to the top of the Windows z-order (.lift() loop) to remain visible during full-screen gameplay.

Draggable & Resizable: Users can click and drag any text on the overlay to reposition it, or drag the bottom-right corner to resize.

Real-Time Telemetry: Tracks Frames Per Second (FPS) via EnumDisplaySettingsW, alongside GPU Temperature, Clock Speed, and Memory Usage.

Dynamic Thresholding: Metrics dynamically change color (Green = Optimal, Orange = Caution, Red = Danger) based on hardcoded limits.

Privacy-First & Zero-Trust

The application contains absolutely zero network calls, telemetry beacons, or third-party wrappers. The system reads solely from local kernel memory and pre-installed display drivers, ensuring complete isolation and adherence to the Elite Coders Best Practice Bible invariants.
