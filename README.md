# Xterilize

Real-time surgical contamination detection using MR body tracking.

MR 바디 트래킹 기반 수술실 오염 감지 시스템

[![Demo](https://img.shields.io/badge/Demo-YouTube-red)](https://www.youtube.com/@aaappp5789/shorts)

---

## Overview

Minor contamination behaviors in the operating room—hands dropping below the waist, unconsciously touching the face, brushing against non-sterile surfaces—can compromise patient safety. These actions often go unnoticed during high-focus surgical procedures.

수술 중 무의식적 오염 행동(손 위치 하강, 얼굴 접촉, 비멸균 표면 접촉)은 환자 안전을 위협하지만, 고도의 집중이 필요한 수술 중에는 인지하기 어렵습니다.

Xterilize addresses this by leveraging Meta Quest's 84-joint body tracking to monitor surgeon movements in real-time. When contamination risk is detected, the system delivers immediate feedback through peripheral vision and spatial audio—designed to alert without disrupting surgical focus.

84개 관절 추적으로 수술자의 움직임을 실시간 모니터링하고, 주변 시야와 공간 음향을 통해 수술 집중을 방해하지 않으면서 즉각적인 경고를 제공합니다.

**Three contamination categories:**

| Category | Examples |
|:---|:---|
| **Habitual behavior** | Hands below waist, face touching |
| **Non-sterile contact** | Floor, cables, door handles, equipment exterior |
| **Movement-based** | Unintentional contact during posture changes |

---

## Detection

### Habitual Behavior

Meta Movement SDK의 Body Tracking 모듈로 84개 관절 위치·회전 데이터를 실시간 추적합니다.

**Hand position drop**
- Waist height calibrated from pelvis joint (`XR_FULL_BODY_JOINT_HIPS_META`) at device wear time
- Triggered when `hand_y < hip_y - δ` (δ ≈ 3-5cm) persists for 500ms+

**Face contact**
- Head collider (`XR_FULL_BODY_JOINT_HEAD_META`) defines face region
- Calculates minimum distance between fingertip joints and face surface
- Triggered when `d_min < 1.0cm`
- Mask/goggle areas apply penalty weights for heightened sensitivity

### Non-sterile Contact

비멸균 영역(바닥, 장비 외부, 케이블, 문 손잡이 등)은 Unity 씬 내에서 Non-sterile Label Map으로 사전 태깅됩니다.

**Distance-based alert escalation:**

| Distance | Level | Response |
|:---:|:---:|:---|
| d ≤ 5cm | Proximity | Yellow emission |
| d ≤ 1cm | Caution | Red emission + audio |
| d ≈ 0 + v > 0.05m/s | Contact | Full warning + logged |

Distance calculated per frame using `Vector3.Distance()` between hand/tool transforms and tagged surfaces. Velocity vector (`v = Δposition / Time.deltaTime`) distinguishes dynamic contact from static proximity, reducing false positives.

### Movement-based Detection

FSM 패턴으로 오염 빈발 수술 시나리오를 구조화합니다.

```
Idle → Instruction → Action → Evaluation
```

- Unity Animator Controller 기반 NPC 집도의가 상태 전환 및 제스처 애니메이션 수행
- Unity Audio Source로 "수술 도구 전달", "장비 위치 조정" 등 음성 지시 출력
- 실제 수술실 협업 상황을 재현하여 오염 발생 가능성이 높은 동작 유도

**Hand interaction:**
- OVRHand/OVRSkeleton으로 캐릭터 바디 리그에 실시간 본 매핑
- OVRGrabber/OVRGrabbable 기반 그랩 시스템으로 수술 도구 파지
- Fixed Joint로 자연스러운 홀딩 자세, Configurable Joint로 도구 무게감 및 관성 시뮬레이션

---

## Feedback

수술자의 시야와 인지 부하를 최소화하면서 즉각적 피드백을 제공하는 다중 모달 인터페이스입니다.

### Visual

| Element | Implementation |
|:---|:---|
| Peripheral warning | Red vignette at 15-20° FOV edges, central vision unobstructed |
| Object highlight | Shader Graph emission ramp (yellow → red) on non-sterile objects |
| Auto-dismiss | 1-second fade-out via Lerp interpolation |

습관적 행동 감지 시 시야 경계부가 붉은색으로 강조되며, 비멸균 물체 접근 시 해당 오브젝트의 머티리얼 Emission 값이 실시간 변경됩니다.

### Audio

| Feature | Implementation |
|:---|:---|
| Directional panning | L/R stereo based on contamination source position |
| Distance attenuation | Volume scales with proximity to contamination point |
| Danger tone | Low-frequency bass added at danger level |

Unity Audio Source의 3D Spatial Audio로 오염 발생 위치 기준 공간 음향을 렌더링합니다.

### Cognitive Load Management

- Duplicate suppression: Same event ignored for 3 seconds
- Adaptive thresholds: Auto-calibrates to user body size, habits, surgery type

경고 과다 반복으로 인한 집중력 저하를 방지합니다.

---

## Data Logging

C# StreamWriter로 CSV 형식 저장, UTF-8 인코딩, 0.5초 버퍼 플러시 주기로 실시간 데이터 손실을 방지합니다.

```csv
Timestamp,EventType,Position_X,Position_Y,Position_Z,RiskLevel,ResponseTime
1704067200000,HAND_BELOW_HIP,0.12,-0.45,0.31,WARNING,null
1704067201500,FACE_CONTACT,0.08,1.62,0.15,DANGER,820
1704067205000,NONSTERILE_CONTACT,-0.25,0.80,0.42,DANGER,1150
```

Unity 정적 변수로 세션 전반의 누적 오염 횟수, 경고 빈도, 반응 시간 통계를 관리하며, ScriptableObject로 사용자별 설정값과 임계값 프로파일을 영속 저장합니다.

**Analysis applications:**
- Spatial patterns: Contamination-prone zone heatmaps
- Time-series: Fatigue-correlated contamination frequency
- Learning effect: Response time trends across training sessions

---

## Tech Stack

| Category | Technology |
|:---|:---|
| Platform | Meta Quest 3 / Quest Pro |
| Engine | Unity 2022.3 LTS |
| Body Tracking | Meta Movement SDK (84 joints) |
| Hand Tracking | OVRHand, OVRSkeleton |
| Interaction | OVRGrabber, OVRGrabbable, Fixed/Configurable Joint |
| Audio | Unity Spatial Audio (3D panning, attenuation) |
| Rendering | Shader Graph (emission control) |
| Data | C# StreamWriter (CSV), ScriptableObject |

---

## References

- Rutala et al. (2023). *Risk of disease transmission from contaminated surgical instruments*. AJIC
- Sánchez-Margallo et al. (2021). *Application of mixed reality in medical training*. Frontiers in VR
- Yu et al. (2021). *Clean-AR: Using AR for reducing contamination risk*. CDBME
- Meta Platforms (2024). *Body Tracking for Movement SDK*
