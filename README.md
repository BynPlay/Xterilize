# 🏥 SterileGuard MR

**Real-time Contamination Detection and Warning System in Surgical Settings Using Mixed Reality Sensing**

*혼합현실 센싱 기술을 활용한 수술실 오염 감지 및 경고 시스템*

A VR training system that detects contamination behaviors in real-time using MR body tracking technology and provides intuitive multi-modal feedback to maintain sterile conditions in the operating room.

[![Demo Video](https://img.shields.io/badge/Demo-YouTube-red?style=for-the-badge&logo=youtube)](https://www.youtube.com/@aaappp5789/shorts)

---

## 🎯 Problem Statement

Minor contamination behaviors in the operating room can have fatal consequences for patient safety.

*수술실에서의 사소한 오염 행동이 환자 안전에 치명적인 영향을 미칩니다.*

| Type | Examples |
|:---|:---|
| **Habitual Behavior** | Hands dropping below waist, unconscious face touching |
| **Non-sterile Contact** | Floor, equipment exterior, cables, door handles |
| **Movement-based Contact** | Unintentional contact during posture changes or movement |

---

## ✨ Key Features

### 🔍 Real-time Detection
- **84-joint tracking**: Full body tracking based on Meta Movement SDK
- **Distance-based alerts**: Progressive warnings at 5cm (caution) → 1cm (danger)
- **Velocity vector analysis**: Distinguishes dynamic contact from static proximity to minimize false positives

### 🎨 Multi-modal Feedback
- **Visual**: Peripheral vision color changes + object emission effects
- **Audio**: 3D spatial audio for intuitive contamination location/distance awareness

### 📊 Data-driven Learning
- CSV logging for behavior pattern analysis
- Supports personalized training scenario generation

---

## 🔧 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Meta Quest 3 / Pro                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │  Body Tracking  │  │  Hand Tracking  │  │ Spatial Map │ │
│  │   (84 joints)   │  │  (OVRSkeleton)  │  │             │ │
│  └────────┬────────┘  └────────┬────────┘  └──────┬──────┘ │
└───────────┼────────────────────┼─────────────────┼─────────┘
            │                    │                 │
            ▼                    ▼                 ▼
┌─────────────────────────────────────────────────────────────┐
│                    Detection Engine                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Habitual   │  │  Contact    │  │  Movement-based     │ │
│  │  Behavior   │  │  Detection  │  │  Detection          │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
└─────────┼────────────────┼───────────────────┼─────────────┘
          │                │                   │
          └────────────────┼───────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Feedback System                           │
│  ┌─────────────────────┐    ┌─────────────────────────────┐│
│  │   Visual Feedback   │    │     Audio Feedback          ││
│  │  • HUD Edge Color   │    │  • 3D Spatial Sound         ││
│  │  • Object Emission  │    │  • Distance Attenuation     ││
│  │  • Material Change  │    │  • Directional Panning      ││
│  └─────────────────────┘    └─────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Data Logging                             │
│                  CSV Export → Dashboard                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧠 Detection Algorithms

### 1. Habitual Behavior Detection

**Hand Position Drop Detection** *(손 위치 하강 감지)*
```
Condition: Hand_y < Hip_y - δ (δ ≈ 3-5cm)
           AND duration > 500ms
```

**Face Contact Detection** *(얼굴 접촉 감지)*
```
Condition: d_min(Hand_tip, Face_collider) < 1.0cm
           Mask/goggle areas → penalty weight applied
```

### 2. Non-sterile Contact Detection

| Distance | Level | Feedback |
|:---:|:---:|:---|
| d ≤ 5cm | ⚠️ Caution | Yellow emission |
| d ≤ 1cm | 🚨 Danger | Red emission + alpha adjustment |
| d ≈ 0 & v > 0.05m/s | ❌ Contact | Full warning + log recorded |

### 3. Movement-based Detection

FSM-based workflow for structuring contamination-prone scenarios:

*오염 빈발 상황을 시나리오로 구조화*
```
Idle → Instruction → Action → Evaluation → Idle
```

---

## 🎨 Feedback Design

### Visual Feedback
- **Peripheral warning**: Limited to HUD 15-20° area, non-intrusive to central vision
- **Object highlight**: Real-time emission changes via Shader Graph
- **Auto-dismiss**: Fade-out within 1 second to maintain workflow

### Audio Feedback
- **Spatial sound**: Left/right panning based on contamination location
- **Distance attenuation**: High volume nearby ↔ low volume at distance
- **Danger stage**: Low-frequency bass tone to induce alertness

### Cognitive Load Management
- Suppresses duplicate alerts for same event within 3 seconds
- Auto-calibrates thresholds per user (body size, habits, surgery type)

---

## 🛠 Tech Stack

| Category | Technology |
|:---|:---|
| **Platform** | Meta Quest 3 / Quest Pro |
| **Engine** | Unity 2022.3 LTS |
| **SDK** | Meta XR SDK, Movement SDK, Interaction SDK |
| **Tracking** | OVRHand, OVRSkeleton, Body Tracking (84 joints) |
| **Audio** | Unity Spatial Audio, 3D Sound |
| **Interaction** | OVRGrabber, OVRGrabbable, Fixed/Configurable Joint |
| **Data** | C# StreamWriter (CSV), ScriptableObject |

---

## 📊 Data Logging Schema

```csv
Timestamp,EventType,Position_X,Position_Y,Position_Z,RiskLevel,ResponseTime
1704067200000,HAND_BELOW_HIP,0.12,-0.45,0.31,WARNING,null
1704067201500,FACE_CONTACT,0.08,1.62,0.15,DANGER,0.8
1704067205000,NONSTERILE_CONTACT,-0.25,0.80,0.42,DANGER,1.2
```

### Analysis Applications
- **Spatial patterns**: Heatmap of contamination-prone zones
- **Time-series analysis**: Contamination frequency changes with fatigue
- **Learning effect**: Response time reduction trends through repeated training

---

## 🚀 Future Directions

### Near-term
- [ ] Reinforcement learning-based personalized warning strategies
- [ ] Cloud dashboard integration
- [ ] Multi-user collaborative training mode

### Long-term
- [ ] Extension to real operating rooms via Inside-Out Body Tracking
- [ ] Computer vision-based automatic non-sterile zone recognition
- [ ] Real-time sterility monitoring system

---

## 📈 Market Context

The MR healthcare market is projected to grow from **$1.27B (2024) → $67.45B (2034)** at a CAGR of 48.74%.
Surgical planning and simulation is expected to show the highest growth rate.

*MR 헬스케어 시장은 연평균 48.74% 성장 전망. 수술 계획 및 시뮬레이션 분야가 최고 성장률 예상.*

**Real-world Applications**
- Providence Swedish Hospital: 100+ neurosurgery planning cases with HoloLens 2
- National University Hospital Singapore: MR technology in 100+ surgeries across 8-9 specialties

---

## 📚 References

1. Rutala, W. A. et al. (2023). *Risk of disease transmission from contaminated surgical instruments*. AJIC.
2. Sánchez-Margallo, J. A. et al. (2021). *Application of mixed reality in medical training*. Frontiers in VR.
3. Yu, K. et al. (2021). *Clean-AR: Using AR for reducing contamination risk*. CDBME.
4. Meta Platforms. (2024). *Body Tracking for Movement SDK*. Meta Developers.

---

## 📝 License

This project was developed for academic research purposes.
