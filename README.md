# Unity AI Open Beta — 교육 자료 모음

> **Unity AI Open Beta** (2026년 5월 공개) — Unity 6+ 에디터에 내장된 AI 도구 모음

## 목차

| # | 파일 | 난이도 | 설명 |
|---|------|--------|------|
| 1 | [`01_concept-basics.md`](01_concept-basics.md) | 초급 | Unity AI의 핵심 개념 이해 |
| 2 | [`02_setup-installation.md`](02_setup-installation.md) | 초급 | 설치 및 환경 설정 |
| 3 | [`03_ai-assistant-basics.md`](03_ai-assistant-basics.md) | 초급 | AI Assistant 기본 사용법 |
| 4 | [`04_ai-gateway.md`](04_ai-gateway.md) | 중급 | AI Gateway로 외부 모델 연결 |
| 5 | [`05_mcp-server-basics.md`](05_mcp-server-basics.md) | 중급 | MCP Server 개념과 설정 |
| 6 | [`06_intermediate-workflows.md`](06_intermediate-workflows.md) | 중급 | 실전 워크플로우 |
| 7 | [`07_advanced-customization.md`](07_advanced-customization.md) | 고급 | 커스텀 도구 및 Skills 개발 |
| 8 | [`08_best-practices.md`](08_best-practices.md) | 고급 | 모범 사례 & 보안 |
| 9 | [`09_troubleshooting.md`](09_troubleshooting.md) | 참고 | 문제 해결 가이드 |
| 10 | [`10_project-examples.md`](10_project-examples.md) | 종합 | 실전 프로젝트 예제 (기본) |
| 11 | [`project_pick_and_place.md`](project_pick_and_place.md) | 로보틱스 | Niryo One Pick-and-Place (URDF → ROS → MoveIt) |
| 12 | [`project_pose_estimation.md`](project_pose_estimation.md) | 로보틱스 | UR3 Object Pose Estimation (Perception → 딥러닝) |
| 13 | [`project_nav2_slam.md`](project_nav2_slam.md) | 로보틱스 | Turtlebot3 Nav2 SLAM (Warehouse → LiDAR → 매핑) |

## Unity AI 구성 요소

```
┌─────────────────────────────────────────────────┐
│                 Unity AI Suite                   │
├─────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐     │
│  │  AI Assistant     │  │  AI Gateway       │     │
│  │  (에디터 내장 에이전트)│  │  (외부 AI 모델 연결)│     │
│  └──────────────────┘  └──────────────────┘     │
│  ┌──────────────────┐  ┌──────────────────┐     │
│  │  MCP Server      │  │  Generators      │     │
│  │  (IDE-Editor 연결)│  │  (에셋 생성)      │     │
│  └──────────────────┘  └──────────────────┘     │
│  ┌──────────────────────────────────────────┐   │
│  │  Skills (확장 가능한 능력 시스템)            │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

## 교육 순서 추천

- **초급**: 01 → 02 → 03
- **중급**: 04 → 05 → 06
- **고급**: 07 → 08
- **참고**: 09 → 10
- **로보틱스 프로젝트**:
  - `project_pick_and_place.md` — ROS + MoveIt 경로 계획
  - `project_pose_estimation.md` — 합성 데이터 + 딥러닝 Pose 추정
  - `project_nav2_slam.md` — Nav2 SLAM 매핑 및 위치추정

## 플랫폼 비교: Unity AI vs Unreal Engine vs NVIDIA Isaac Sim

### 개요

Unity AI 외에도 Unreal Engine과 NVIDIA Isaac Sim이 로보틱스/AI 개발 환경을 제공합니다. 각 플랫폼의 접근법과 강점을 비교합니다.

### 종합 비교표

| 영역 | Unity AI | Unreal Engine | NVIDIA Isaac Sim |
|------|----------|---------------|-----------------|
| **AI 어시스턴트** | ✅ AI Assistant (공식 내장) | ❌ 없음 (Aura는 서드파티) | ❌ 없음 (MCP로 Claude Code 연동) |
| **MCP Server** | ✅ Unity MCP Server (공식) | ✅ UE 5.8 MCP Plugin (실험적, 2026.6) | ✅ Isaac Sim MCP (6.0 GA 정식) |
| **AI Gateway** | ✅ AI Gateway (공식) | ❌ 없음 (GenAI Plugin 서드파티) | ❌ 없음 (NVIDIA NIM 직접 연동) |
| **렌더링** | URP / HDRP | Nanite / Lumen | **Omniverse RTX** (Path Tracing) |
| **비주얼 스크립팅** | ❌ (C# 전용) | ✅ **Blueprints** | ✅ **OmniGraph** |
| **개발 언어** | C# | C++ / Blueprints | **Python** + Core API |
| **로보틱스 URDF** | ✅ URDF-Importer (공식) | ❌ 없음 (서드파티) | ✅ **URDF/MJCF** 기본 내장 |
| **ROS 2 연동** | ✅ ROS-TCP-Connector (공식) | ❌ 없음 | ✅ **ROS 2 Bridge** 내장 |
| **강화학습(RL)** | ❌ (ML-Agents deprecated) | ❌ | ✅ **Isaac Lab** (수천 병렬 환경) |
| **합성 데이터** | ✅ Perception Package | ❌ | ✅ **Isaac Replicator** + **Cosmos** |
| **파운데이션 모델** | ❌ (외부 모델 Gateway 연결) | ❌ | ✅ **Cosmos WFM** (물리 기반 세계 모델) |
| **대규모 물리** | PhysX (게임 목적) | Chaos Physics | **PhysX + Newton** (결정론, 산업용) |
| **GPU 요구사항** | 일반 GPU | 일반 GPU | **RTX GPU 필수** |
| **라이선스** | Unity Pro/Enterprise | UE 라이선스 | **오픈소스** (GitHub) |

### 상세 분석

#### Unreal Engine (UE 5.8+)

| 강점 | 한계 |
|------|------|
| Nanite/Lumen 최고 수준 비주얼 | AI 도구가 **산발적** - 공식 AI Assistant 없음 |
| Blueprints로 비개발자도 접근 가능 | 로보틱스 **전용 패키지 없음** (URDF, ROS 등) |
| 방대한 게임 에코시스템 | MCP Plugin이 **실험적** 단계 (UE 5.8, 2026.6월 출시) |
| C++ 성능 | AI 생성 코드가 많을수록 Unreal의 복잡성이 부담 |

> **UE 5.8 MCP Plugin**: Claude Code/Codex CLI가 에디터 제어 가능하나 아직 인증, Toolset, 안정성 미흡. 로보틱스보다는 게임 에셋 배치/조명에 특화.

#### NVIDIA Isaac Sim (6.0 GA)

| 강점 | 한계 |
|------|------|
| **로보틱스를 위해 태어난 플랫폼** — 게임 엔진 확장이 아님 | **RTX GPU 필수** — 일반 GPU에서 실행 불가 |
| OmniGraph (Blueprints) + Python (C#) 동시 제공 | USD 학습 곡선 (복잡한 데이터 모델) |
| **Isaac Lab**: 수천 개 병렬 RL 환경, Sim-to-Real | 게임 개발 **불가** — 순수 로보틱스 전용 |
| **Cosmos WFM**: 실제 영상 → 물리 기반 합성 데이터 | 커뮤니티 규모가 Unity/Unreal 대비 작음 |
| **Newton + PhysX** 이중 물리 백엔드 (결정론/성능 선택) | NVIDIA 생태계 종속 |
| **MCP Server + Skills** 정식 지원 (6.0 GA) | |

### 선택 가이드

| 목적 | 추천 플랫폼 | 이유 |
|------|-----------|------|
| **게임 + AI** | Unity AI | Assistant, Gateway, MCP 원스톱 |
| **AAA 게임 + AI** | Unreal + UE 5.8 MCP | Blueprints + 고퀄리티 비주얼 |
| **로보틱스 시뮬레이션 + AI** | **Isaac Sim** | URDF/ROS/RL/합성데이터 전부 내장 |
| **산업용 디지털 트윈** | **Isaac Sim** + Unity | Isaac 물리 + Unity AI Assistant 조합 |
| **휴머노이드 로봇** | **Isaac Sim + GR00T** | NVIDIA 전용 파이프라인 |
| **ROS 2 개발** | Unity / Isaac Sim | 둘 다 ROS 2 지원 (Unreal은 미지원) |

### 요약

- **Unity AI**: "게임 개발자가 AI로 생산성을 높이는" 데 최적화. AI Assistant, Gateway, MCP가 하나의 패키지로 통합.
- **Unreal Engine**: Blueprints의 강력함 + UE 5.8 MCP로 AI 연결 시작. 로보틱스는 **비어 있는 영역**.
- **Isaac Sim**: 처음부터 로보틱스 + Physical AI를 위해 설계. Unreal의 비주얼 + Blueprints, Unity의 접근성 + AI를 결합하고, NVIDIA GPU/Cosmos/Isaac Lab의 수직 통합이 차별점.

## 참고 링크

### Unity AI
- [Unity AI 공식 페이지](https://unity.com/features/ai)
- [Unity AI 공식 문서](https://docs.unity3d.com/Packages/com.unity.ai.assistant@latest)
- [Unity AI 시작 가이드](https://support.unity.com/hc/en-us/articles/48060149523476)
- [Unity MCP Server 가이드](https://unity.com/blog/unity-ai-mcp-how-to-get-started)

### 플랫폼별 AI/MCP
- [Unreal Engine 5.8 MCP Plugin (실험적)](https://dev.epicgames.com/documentation/unreal-engine/unreal-mcp-in-unreal-editor)
- [Unreal MCP Server (ChiR24, 커뮤니티)](https://github.com/ChiR24/Unreal_mcp)
- [UE-MCP (db-lyon, 커뮤니티)](https://github.com/db-lyon/ue-mcp)
- [NVIDIA Isaac Sim 6.0](https://developer.nvidia.com/isaac/sim)
- [Isaac Sim MCP Server](https://forums.developer.nvidia.com/t/isaac-sim-6-0-general-availability/372621)
- [NVIDIA Cosmos World Foundation Model](https://www.nvidia.com/en-us/ai/cosmos)

### 로보틱스 프로젝트
- [Unity-Robotics-Hub — Pick-and-Place](https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/pick_and_place/README.md)
- [Robotics-Object-Pose-Estimation](https://github.com/Unity-Technologies/Robotics-Object-Pose-Estimation)
- [Robotics-Nav2-SLAM-Example](https://github.com/Unity-Technologies/Robotics-Nav2-SLAM-Example)
- [URDF-Importer](https://github.com/Unity-Technologies/URDF-Importer)
- [ROS-TCP-Connector](https://github.com/Unity-Technologies/ROS-TCP-Connector)
- [Unity Perception Package](https://github.com/Unity-Technologies/com.unity.perception)
- [Robotics Warehouse](https://github.com/Unity-Technologies/Robotics-Warehouse)
- [Aura AI Agent for Unreal](https://www.tryaura.dev/)
- [Ludus AI Toolkit for Unreal](https://ludusengine.com/)
- [GenAI for Unreal Plugin](https://www.fab.com/listings/abc123)
