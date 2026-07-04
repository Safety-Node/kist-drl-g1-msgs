# g1_onboard_msgs

**PC ↔ NX 인터페이스 타입의 단일 진실 소스(SSOT).**

이 repo는 KIST-DRL-G1 시연 시스템의 ROS 2 커스텀 메시지 패키지(`g1_onboard_msgs`) 하나만 담습니다.
로직은 없습니다 — `.msg` 정의뿐입니다. `kist-drl-g1-workstation`(PC)과 `kist-drl-g1-onboard`(NX)가
**둘 다 이 repo를 git submodule로 참조**해서, 양쪽이 항상 **동일한 메시지 정의 = 동일한 DDS type hash**를
빌드하도록 보장합니다.

> 이 repo가 생기기 전에는 `.msg` 파일이 양 repo에 각각 복제되어 있었고, 실제로 `AudioPCM.msg`의
> 상수(`SAMPLE_RATE`/`CHANNELS`/`BIT_DEPTH`)가 한쪽에만 존재하는 drift가 발생했습니다. submodule 단일화로
> 이런 조용한 불일치를 원천 차단합니다.

## 포함 메시지
| 메시지 | 용도 |
|---|---|
| `JointCmd` | 관절 명령 1-step (arm/low) |
| `JointCmdChunk` | VLA action chunk 래퍼 (PC→NX 와이어) |
| `AudioPCM` | 16kHz/16-bit/mono PCM (mic·speaker·TTS) |
| `BufState` | motor_controller 링버퍼 텔레메트리 |
| `LocoCommand` | 이산 자세 전환 (StandUp/SitDown/…) |
| `EstopFlag` | safety_monitor E-STOP (reason enum 0–8) |
| `SpeakerState` | speaker 재생 상태 (echo-cancel 힌트) |

## 소비 repo에서 사용법 (submodule)
```bash
# 최초 clone 시
git clone --recursive <consumer-repo-url>
# 또는 이미 clone 했다면
git submodule update --init --recursive

# 빌드 (colcon 이 submodule 경로의 패키지를 자동 인식)
colcon build --packages-select g1_onboard_msgs
```

submodule 경로:
- `kist-drl-g1-onboard`   → `src/g1_onboard_msgs`
- `kist-drl-g1-workstation` → `ros2_ws/src/g1_onboard_msgs`

## 계약(인터페이스) 변경 워크플로 — 원자적으로
1. 이 repo에서 `.msg` 수정 → PR → 머지 → **버전 태그**(예: `v0.2.0`).
2. **양 소비 repo가 같은 작업 세트에서** submodule 포인터를 새 커밋으로 동시 bump:
   ```bash
   cd <consumer>/…/g1_onboard_msgs && git fetch && git checkout v0.2.0
   cd <consumer-root> && git add <submodule-path> && git commit -m "chore(msgs): bump g1_onboard_msgs to v0.2.0"
   ```
3. 두 소비 repo가 항상 **같은 태그**를 가리키는지 CI에서 검증(권장).

> `.msg`의 **필드/타입**을 바꾸면 DDS type hash가 바뀌어 구/신 노드가 서로 안 붙습니다. 반드시 양쪽을 함께 bump하세요.
> (상수만 추가/변경은 hash 불변이지만, 코드가 그 상수를 참조하면 여전히 양쪽을 맞춰야 합니다.)

## 버전
- `v0.1.0` — `kist-drl-g1-onboard` 에서 히스토리 보존 추출한 최초 버전 (권위본).
