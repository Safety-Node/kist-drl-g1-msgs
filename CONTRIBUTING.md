# Contributing to kist-drl-g1-msgs

This repository is the **single source of truth** for the PC ↔ NX wire interface.
A change here ripples into **both** consumer repos (`kist-drl-g1-workstation`,
`kist-drl-g1-onboard`). Treat every change as a cross-repo contract change.

## Branch & PR conventions (mirror the consumer repos)
Enforced by the `pr-meta` workflow (required checks: `branch-name`, `pr-title`).

- **Branch name**: `TASK-{number}` (Notion-linked work) or `chore/{description}`.
- **PR title**: `([TASK-{number}] | [chore]) <type>(<scope>)?: <lowercase subject>`
  where `<type>` ∈ `feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert`.
  - e.g. `[TASK-58] feat(msgs): add LidarHealth message`
  - e.g. `[chore] docs(readme): clarify submodule path`
- **Squash-and-merge only** — the PR title becomes the commit on `main`.

## Versioning (semantic, wire-aware)
Tag every `main` state that consumers should adopt: `vMAJOR.MINOR.PATCH` (annotated tag).

| Bump | When | DDS type hash |
|---|---|---|
| **PATCH** (`v0.1.0 → v0.1.1`) | comments, docs, whitespace — no serialized change | unchanged |
| **MINOR** (`v0.1.x → v0.2.0`) | add a **new message type**, or add a **constant** to an existing message | unchanged for existing msgs |
| **MAJOR** (`v0.x → v1.0.0`) | change/rename/reorder/remove a **field** in an existing message, or delete a message | **CHANGES** — old⇄new nodes stop communicating |

> ⚠️ Adding a **field** (not a constant) to an existing message also changes that
> message's type hash. Even though it feels additive, treat field additions as a
> **MAJOR/breaking** change for wire compatibility and coordinate both consumers.

## The golden rule — coordinate both consumers
Any release that changes the wire (a MINOR that adds a field, or any MAJOR) **must**
land together with submodule-bump PRs in **both** consumer repos:

1. Merge the change here, then tag it (`git tag -a vX.Y.Z -m "..."; git push origin vX.Y.Z`).
2. In **each** consumer repo, on a `chore/bump-g1-msgs-vX-Y-Z` branch:
   ```bash
   cd <submodule-path> && git fetch --tags && git checkout vX.Y.Z && cd -
   git add <submodule-path>
   git commit -m "[chore] chore(msgs): bump g1_onboard_msgs to vX.Y.Z"
   ```
3. In this PR's description, link **both** consumer PRs. Reviewers merge all three
   together (or in immediate succession). Never bump only one consumer.

A constants-only change does not break the hash, but if any consumer *reads* the
new constant it still needs the bump — so coordinate anyway.

## Local checks before opening a PR
```bash
mkdir -p /tmp/ws/src && ln -s "$PWD" /tmp/ws/src/g1_onboard_msgs
cd /tmp/ws && source /opt/ros/humble/setup.bash && colcon build
source install/setup.bash && ros2 interface list | grep g1_onboard_msgs
```

## Review
Interface changes need approval from the interface owner. Add a `CODEOWNERS`
file with the owner's GitHub handle and enable branch protection requiring:
`branch-name`, `pr-title`, and `build` status checks + 1 code-owner review.
