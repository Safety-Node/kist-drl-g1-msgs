# kist-drl-g1-msgs

**Single source of truth (SSOT) for the PC ↔ NX interface types.**

This repository contains exactly one thing: the ROS 2 custom message package
**`g1_onboard_msgs`** for the KIST-DRL-G1 demo system. No logic — just `.msg`
definitions. Both `kist-drl-g1-workstation` (PC) and `kist-drl-g1-onboard` (NX)
consume it **as a git submodule**, so both build the *same* message definitions
and therefore the *same* DDS type hashes. This makes cross-machine type drift
structurally impossible.

> Repo name (`kist-drl-g1-msgs`) ≠ ROS package name (`g1_onboard_msgs`). That is
> intentional and fine: the package name comes from `package.xml`, and the
> submodule is checked out into a directory named `g1_onboard_msgs` on each side.

> Before this repo existed, the `.msg` files were copy-pasted into both consumer
> repos and had already drifted (the `AudioPCM.msg` constants existed on only one
> side). Sharing one submodule removes that entire class of silent bug.

## Messages
| Message | Purpose |
|---|---|
| `JointCmd` | Single-step joint command (arm / low) |
| `JointCmdChunk` | VLA action-chunk wrapper (PC→NX wire) |
| `AudioPCM` | 16 kHz / 16-bit / mono PCM (mic, speaker, TTS) |
| `BufState` | motor_controller ring-buffer telemetry |
| `LocoCommand` | Discrete posture transitions (StandUp / SitDown / …) |
| `EstopFlag` | safety_monitor E-STOP (reason enum 0–8) |
| `SpeakerState` | Speaker playback state (echo-cancel hint) |

## Using it in a consumer repo (submodule)
```bash
# Fresh clone
git clone --recursive <consumer-repo-url>
# Existing clone
git submodule update --init --recursive

# Build (colcon discovers the package at the submodule path automatically)
colcon build --packages-select g1_onboard_msgs
```

Submodule paths:
- `kist-drl-g1-onboard`     → `src/g1_onboard_msgs`
- `kist-drl-g1-workstation` → `ros2_ws/src/g1_onboard_msgs`

Consumer *code does not change*: the package name is still `g1_onboard_msgs`, so
existing `from g1_onboard_msgs.msg import ...` and `<depend>g1_onboard_msgs</depend>`
keep working. Only the *source location* changes (submodule instead of a local copy).

## Changing the contract
See [CONTRIBUTING.md](CONTRIBUTING.md). In short: `.msg` field/type changes break the
DDS type hash, so every wire-affecting release is tagged (`vX.Y.Z`) and adopted by
**both** consumers in coordinated submodule-bump PRs. Never bump one side alone.

## Versions
- `v0.1.0` — initial extraction from `kist-drl-g1-onboard` (history preserved).
