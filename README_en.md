<div align="center">
  <img src="https://avatars.githubusercontent.com/u/245985800?s=200&v=4" alt="FasterEdge logo" width="100" />
  <h2>FasterEdge Documentation Center</h2>
  <h3>Public Documentation for the FasterEdge Framework and Ecosystem Components</h3>
</div>

### 1. Introduction

- **Documentation baseline**: `1.0.20260831`
- **Purpose**: this repository centrally maintains public documentation for the FasterEdge core framework and its supporting components.
- **Source of truth**: each component's source repository remains authoritative for implementation details, parameters and versions. This repository provides cross-repository navigation, architecture descriptions, deployment guides, security recommendations and composition examples.

### 2. Quick Navigation

| Component | Purpose | Documentation | Source |
|---|---|---|---|
| FasterEdge | Core Atom / Data / Ability / Command / Transport framework | [Documentation](FasterEdge/README.md) | [Repository](https://github.com/FasterEdge/FasterEdge) |
| DontCrack | Cross-platform process supervision, automatic restart, health probes and observability | [Documentation](DontCrack/README.md) | [OpenHarmony](https://github.com/FasterEdge/DontCrack4OpenHarmonyLinuxKernelSide) · [Android](https://github.com/FasterEdge/DontCrack4AndroidLinuxKernelSide) · [Linux](https://github.com/FasterEdge/DontCrack4ManyLinux) · [Windows](https://github.com/FasterEdge/DontCrack4Windows) |
| ProxyArea | General-purpose HTTP request forwarding, destination allowlist and TLS | [Documentation](ProxyArea/README.md) | [Repository](https://github.com/FasterEdge/ProxyArea) |
| SimpleWebShell | WebShell with sessions and file transfer | [Documentation](SimpleWebShell/README.md) | [Repository](https://github.com/FasterEdge/SimpleWebShell) |
| SimpleTimeService | Local/NTP UTC time service | [Documentation](SimpleTimeService/README.md) | [Repository](https://github.com/FasterEdge/SimpleTimeService) |
| TsnHub | Design for local IPC and OPC UA/TSN bridging | [Documentation](TsnHub/README.md) | [Repository](https://github.com/FasterEdge/TsnHub) |
| Example | Example-repository status and available demo entry points | [Documentation](Example/README.md) | [Repository](https://github.com/FasterEdge/Example) |
| FasterEdgeDoctor | Local/remote repository and runtime health diagnostics | — | [Repository](https://github.com/FasterEdge/FasterEdgeDoctor) |
| MCU / FPGA Ports | Hardware implementations for Arduino, PlatformIO, Keil, MounRiver, Vivado and other toolchains | — | [Organization repositories](https://github.com/FasterEdge) |
| RelayNode | SW2MQTT and SW2USB hardware node projects | — | [Organization repositories](https://github.com/FasterEdge) |

### 3. Recommended Reading Order

1. [Ecosystem](00-overview/ecosystem.md)
2. [Overall Architecture](00-overview/architecture.md)
3. [Versioning Strategy](00-overview/versioning.md)
4. [Security Baseline](00-overview/security.md)
5. [FasterEdge Quick Start](FasterEdge/README.md)
6. [DontCrack Process Supervision](DontCrack/README.md)
7. [ProxyArea General-purpose Proxy](ProxyArea/README.md)

### 4. Documentation Scope

- **Public content only**: this repository contains public technical documentation only. It does not duplicate patents, software-copyright materials, registration documents or internal information from `PrivateDocument`.
- **Placeholder examples**: keys, addresses and paths shown in examples are placeholders and must not be used directly in production.
- **Current baseline**: historical versions in README files are retained only as change records; the unified current runtime baseline is `1.0.20260831`.

### 5. Maintenance Principles

- **Update order**: update the component source and component README first, then synchronize this repository.
- **Verifiability**: every parameter, endpoint and command name must be traceable to source code.
- **Implementation status**: functionality absent from the current source must be explicitly labeled as a design goal or pending work, never as completed.
- **Repository hygiene**: do not commit binaries, archives, certificates, keys, logs, IDE metadata or operating-system metadata.
