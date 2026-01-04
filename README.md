# 🏥 Xterilize

**Real-time Surgical Contamination Detection System Using Mixed Reality Body Tracking**

*MR 바디 트래킹 기반 수술실 오염 행동 실시간 감지 시스템*

A VR training system that leverages 84-joint body tracking to detect contamination behaviors and deliver non-intrusive multi-modal warnings, helping medical staff maintain sterile protocols.

[![Demo](https://img.shields.io/badge/▶_Demo-YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@aaappp5789/shorts)
![Unity](https://img.shields.io/badge/Unity-2022.3_LTS-000000?style=for-the-badge&logo=unity)
![Meta Quest](https://img.shields.io/badge/Meta_Quest_3-0467DF?style=for-the-badge&logo=meta)

---

## Problem

Operating room contamination from unconscious behaviors—hands dropping below waist, touching face, brushing non-sterile surfaces—poses serious risks to patient safety. Traditional training methods lack real-time feedback mechanisms.

*수술실에서 무의식적 오염 행동(손 위치 하강, 얼굴 접촉, 비멸균 표면 접촉)은 환자 안전에 심각한 위험을 초래합니다. 기존 훈련 방식은 실시간 피드백이 부재합니다.*

---

## Solution

Xterilize uses Meta Movement SDK's 84-joint body tracking to monitor surgeon movements in real-time, detecting three categories of contamination risk and providing immediate feedback without disrupting surgical focus.

*84개 관절 추적으로 세 가지 오염 위험을 실시간 모니터링하고, 수술 집중을 방해하지 않는 즉각적 피드백을 제공합니다.*

---

## Design Principles

*설계 원칙*

| Principle | Implementation |
|:---|:---|
| **Non-intrusive feedback** | Warnings in 15-20° peripheral vision only; central FOV preserved |
| **Progressive escalation** | 3-tier alert system (proximity → caution → danger) prevents alarm fatigue |
| **Context-aware detection** | Velocity vector analysis distinguishes intentional movement from accidental contact |
| **Personalization** | Auto-calibrating thresholds based on user body metrics and behavior patterns |

---

## System Overview

*시스템 구조*

```
                            ┌─────────────────────┐
                            │   Meta Quest 3/Pro  │
                            │                     │
                            │  • Body Tracking    │
                            │  • Hand Tracking    │
                            │  • Spatial Mapping  │
                            └──────────┬──────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                  ▼
          ┌─────────────────┐ ┌───────────────┐ ┌─────────────────┐
          │    Habitual     │ │   Non-sterile │ │   Movement      │
          │    Behavior     │ │    Contact    │ │    Context      │
          └────────┬────────┘ └───────┬───────┘ └────────┬────────┘
                   │                  │                  │
                   └──────────────────┼──────────────────┘
                                      ▼
                            ┌─────────────────────┐
                            │   Feedback Engine   │
                            │                     │
                            │  Visual │ Spatial   │
                            │   HUD   │  Audio    │
                            └──────────┬──────────┘
                                       │
                                       ▼
                            ┌─────────────────────┐
                            │    Data Logger      │
                            │    (CSV Export)     │
                            └─────────────────────┘
```

---

## Detection Logic

*탐지 로직*

### Habitual Behavior — *습관적 행동*

Detects unconscious movements that break sterile protocol.

| Behavior | Detection Rule | Threshold |
|:---|:---|:---|
| Hand below waist | `Hand_y < Hip_y - δ` | δ = 3-5cm, duration > 500ms |
| Face contact | `distance(fingertip, face_collider)` | < 1.0cm |

> Mask and goggle regions apply penalty weights for heightened sensitivity.
> 
> *마스크/고글 영역은 가중치를 적용하여 민감도를 높입니다.*

### Non-sterile Contact — *비멸균 접촉*

Pre-labeled surfaces (floor, cables, door handles) trigger distance-based alerts.

| Distance | Alert Level | Response |
|:---:|:---:|:---|
| ≤ 5cm | ⚠️ Proximity | Yellow emission glow |
| ≤ 1cm | 🔶 Caution | Red emission + audio cue |
| ≈ 0cm | 🚨 Contact | Full warning + event logged |

> Velocity check (`v > 0.05 m/s`) filters out false positives from static postures.
>
> *속도 벡터 검증으로 정적 자세에서의 오탐을 방지합니다.*

### Movement Context — *이동 상황*

FSM-driven surgical scenarios simulate high-risk moments.

```
Idle → Instruction → Action → Evaluation → Idle
```

NPC surgeons issue voice commands ("Pass the scalpel") while user navigates contamination-prone interactions.

*NPC 집도의가 음성 지시를 내리며, 사용자는 오염 위험이 높은 상호작용을 수행합니다.*

---

## Feedback System

*피드백 시스템*

### Visual Design

| Element | Behavior |
|:---|:---|
| **Peripheral HUD** | Red vignette at screen edges; auto-fades in 1s |
| **Object highlight** | Shader Graph emission ramp (yellow → red) |
| **Material restore** | Lerp interpolation back to original state |

> Central vision remains unobstructed to preserve surgical focus.
>
> *중심 시야는 방해하지 않아 수술 집중을 유지합니다.*

### Spatial Audio

| Feature | Purpose |
|:---|:---|
| **Directional panning** | Indicates contamination source location (L/R) |
| **Distance attenuation** | Closer = louder; provides implicit proximity sense |
| **Bass tone on danger** | Low-frequency rumble triggers heightened alertness |

### Cognitive Load Management

- **Debounce**: Same event suppressed for 3 seconds
- **Adaptive thresholds**: Calibrates to user body size, experience level, surgery type

---

## Tech Stack

| Layer | Technologies |
|:---|:---|
| **Platform** | Meta Quest 3 / Quest Pro |
| **Engine** | Unity 2022.3 LTS |
| **Tracking** | Meta Movement SDK (84 joints), OVRHand, OVRSkeleton |
| **Interaction** | OVRGrabber, Fixed Joint, Configurable Joint |
| **Audio** | Unity Spatial Audio (3D panning, attenuation) |
| **Rendering** | Shader Graph (emission control) |
| **Data** | C# StreamWriter → CSV, ScriptableObject |

---

## Data Schema

*데이터 스키마*

```csv
Timestamp,EventType,Position_X,Position_Y,Position_Z,RiskLevel,ResponseTime_ms
1704067200000,HAND_BELOW_HIP,0.12,-0.45,0.31,WARNING,null
1704067201500,FACE_CONTACT,0.08,1.62,0.15,DANGER,820
1704067205000,NONSTERILE_CONTACT,-0.25,0.80,0.42,DANGER,1150
```

**Applications**: Contamination heatmaps · Fatigue-correlated trends · Training progress tracking

*활용: 오염 히트맵 · 피로도 상관분석 · 훈련 진척도 추적*

---

## Roadmap

| Phase | Goals |
|:---|:---|
| **v1.0** | Core detection + feedback system ✅ |
| **v1.1** | Cloud dashboard, multi-user mode |
| **v2.0** | Inside-Out tracking for physical OR environments |
| **v3.0** | CV-based automatic non-sterile zone labeling |

---

## Market Context

MR healthcare market: **$1.27B (2024) → $67.45B (2034)** — CAGR 48.74%

*MR 헬스케어 시장: 연평균 48.74% 성장 전망*

| Institution | Application |
|:---|:---|
| Providence Swedish Hospital | 100+ neurosurgery cases with HoloLens 2 |
| NUH Singapore | MR in 100+ surgeries across 8-9 specialties |

---

## References

1. Rutala et al. (2023). *Risk of disease transmission from contaminated surgical instruments*. AJIC
2. Sánchez-Margallo et al. (2021). *Application of mixed reality in medical training*. Frontiers in VR
3. Yu et al. (2021). *Clean-AR: Using AR for reducing contamination risk*. CDBME
4. Meta Platforms (2024). *Body Tracking for Movement SDK*. Meta Developers
