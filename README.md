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

## 참고 링크

### Unity AI
- [Unity AI 공식 페이지](https://unity.com/features/ai)
- [Unity AI 공식 문서](https://docs.unity3d.com/Packages/com.unity.ai.assistant@latest)
- [Unity AI 시작 가이드](https://support.unity.com/hc/en-us/articles/48060149523476)
- [Unity MCP Server 가이드](https://unity.com/blog/unity-ai-mcp-how-to-get-started)

### 로보틱스 프로젝트
- [Unity-Robotics-Hub — Pick-and-Place](https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/pick_and_place/README.md)
- [Robotics-Object-Pose-Estimation](https://github.com/Unity-Technologies/Robotics-Object-Pose-Estimation)
- [Robotics-Nav2-SLAM-Example](https://github.com/Unity-Technologies/Robotics-Nav2-SLAM-Example)
- [URDF-Importer](https://github.com/Unity-Technologies/URDF-Importer)
- [ROS-TCP-Connector](https://github.com/Unity-Technologies/ROS-TCP-Connector)
- [Unity Perception Package](https://github.com/Unity-Technologies/com.unity.perception)
- [Robotics Warehouse](https://github.com/Unity-Technologies/Robotics-Warehouse)
