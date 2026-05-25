# HwBridge

HwBridge is a Windows C++ hardware bridge for ATM/kiosk flows. It exposes hardware operations over TCP while isolating vendor SDK calls in dedicated worker processes.

## What It Does

- Receives JSON commands from a main app over TCP.
- Routes commands by domain (`CS*`, `Primacy*`, biometric, smart card, USB/print, control).
- Executes hardware calls inside worker processes (not in the server thread).
- Returns framed JSON responses with a stable compatibility contract.
- Supports headless operation and optional local ImGui UI (Dev mode).

## Screen Shots

<img width="944" height="708" alt="3" src="https://github.com/user-attachments/assets/076effeb-ba0d-4363-a158-874afbe001ee" />
<img width="946" height="708" alt="4" src="https://github.com/user-attachments/assets/45ad15e4-2461-4ae0-9262-b91e1ef6c5cb" />

## Architecture (Current)

- Supervisor: `main.cpp`
- TCP server/session: `server/server.cpp`, `server/clienthandler.cpp`
- Command router/contracts: `common/commands/*`
- Domain handlers: `common/handlers/*`
- Worker host/modes: `workers/WorkerModes.cpp`
- Device modules: `modules/*`

Biometric devices currently supported:
- Futronic: `FingerScan`
- Aratek: `AFingerScan`
- Suprema: `SFingerScan`

## Command Contract Summary

- Transport: TCP IPv4
- Inbound accepted:
  - plain JSON
  - 8-digit length-prefixed JSON
- Outbound:
  - always framed JSON (8-digit prefix + JSON body)
- Primary request envelope:

```json
{"DeviceName":"AtmEngine","Command":"<Command>","isReq":true,"TimeOut":60,"ExtraData":"","ExtraData1":""}
```

Full command catalog and real one-line request/response samples:
- `docs/HwBridge_Test_Commands.md`

## Build (Windows)

### Prerequisites

- Visual Studio 2022 C++ toolchain (x86 target)
- CMake 3.23+
- Ninja
- `vcpkg` installed at `C:\vcpkg-master` (as referenced by `CMakeLists.txt`)

### Build Commands

```powershell
cmake -S . -B HwBridgeRelease -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build HwBridgeRelease --target HwBridge -j 8
```

Output binary:
- `HwBridgeRelease/HwBridge.exe`

## Runtime Configuration

Main config file:
- `m_config.ini` (next to executable)

Important areas:
- `[Network]` (listen port)
- `[Features]` (queue mode, worker prewarm flags)
- `[TimeoutPolicy]` (per-command default/cap timeouts)
- `[WorkerHealth]` (worker heartbeat/restart supervision)

Runtime asset/deployment manifest:
- `docs/HwBridge_Runtime_Manifest.md`

## UI Tools

When `DevMode=1`, the local panel includes:
- Dashboard worker/device health summary.
- Operations monitor.
- Registry/config tools.
- Command simulator.
- Device Tests tab with one-click smoke tests for scanner, card, biometric, Evolis, USB, and control commands.

## Modes

- Supervisor mode: default launch (`HwBridge.exe`)
- Worker modes (internal use):
  - `--checkscanner-worker`
  - `--ncard-worker`
  - `--bio-worker`
  - `--suprima-worker`
  - `--evolis-worker`
  - `--ops-worker`

## Documentation

- Full technical documentation: `docs/HwBridge_Full_Documentation.md`
- Command compatibility + examples: `docs/HwBridge_Test_Commands.md`
- Runtime/deployment manifest: `docs/HwBridge_Runtime_Manifest.md`
- Smoke/regression guide: `docs/HwBridge_Smoke_Test.md`
- Migration/status plan: `docs/HwBridge_Supervisor_Zero_Regression_Migration_Plan.md`
- Docs index: `docs/README.md`

## Development Rule

Keep the main app in/out contract stable unless an explicit protocol cutover is planned and approved.
