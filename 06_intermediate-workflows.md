# 06. 중급: 실전 워크플로우

## 6.1 워크플로우 개요

이 장에서는 Unity AI를 실제 개발 파이프라인에 통합하는 방법을 다룹니다.

## 6.2 워크플로우 1: 프로토타이핑 가속

### 목표
아이디어를 빠르게 프로토타입으로 전환

### 프로세스

```
1. 개념 설명 → AI가 씬 구조 생성
2. Plan Mode로 검토
3. 필요한 스크립트 생성
4. 기본 에셋 (placeholder) 생성
5. 플레이 테스트 및 피드백 → AI가 수정
```

### 실제 예시

```
사용자: "2D 플랫포머 게임을 만들어줘. 
         - 왼쪽/오른쪽 이동, 점프
         - 적은 좌우로 이동
         - 동전 수집 시스템
         - 간단한 UI (점수, 목숨)"

→ AI가 10분 내로 기본 구조 완성
```

### 산출물

| 항목 | AI 생성 | 설명 |
|------|---------|------|
| 씬 | MainScene | 플레이어, 바닥, 장애물 배치 |
| 스크립트 | PlayerController.cs | 이동, 점프 |
| 스크립트 | EnemyPatrol.cs | 적 AI |
| 스크립트 | CoinCollectible.cs | 수집 시스템 |
| 스크립트 | GameManager.cs | 게임 상태 관리 |
| UI | Canvas | 점수, 목숨 표시 |
| 에셋 | Placeholder | 큐브/스프라이트 (Generators) |

## 6.3 워크플로우 2: 버그 수정 자동화

### 목표
콘솔 에러 → 자동 분석 → 수정 → 검증

### 프롬프트 예시

```
"콘솔에 있는 모든 에러를 분석해서 수정해줘.
Plan Mode를 켜고, 각 수정 사항을 승인받고 진행해줘"
```

### AI 동작

1. `get_console_logs`로 에러 수집
2. 각 에러의 스택 트레이스 분석
3. 관련 소스 코드 검색
4. 수정안 작성
5. Plan Mode로 제시
6. 승인 후 적용
7. 재테스트로 검증

### 효율성

| 항목 | 수동 | AI 활용 |
|------|------|---------|
| NullReferenceException 추적 | 5-15분 | 1-2분 |
| 씬 설정 오류 수정 | 3-10분 | 30초 |
| 빌드 에러 해결 | 5-20분 | 2-5분 |

## 6.4 워크플로우 3: MCP를 활용한 IDE-Editor 연동

### 설정
- IDE: VS Code + Claude Code
- Unity: MCP Server 활성화
- 연결: stdio 모드

### 실제 세션 예시

```bash
# Claude Code 터미널
# "Unity 씬을 분석하고 최적화할 부분을 찾아줘"
```

```
Claude Code → MCP get_scene_hierarchy() → 씬 구조 확인
Claude Code → MCP get_console_logs() → 경고/에러 확인
Claude Code → 분석 결과 제시
```

### 반복 작업 자동화

```bash
# 에셋 이름 일괄 변경
"Find all GameObjects with 'Temp' prefix and rename them to 'Prop_'"

# 컴포넌트 일괄 추가
"Add a BoxCollider to all GameObjects that don't have one"

# 머티리얼 일괄 변경
"Change all materials to use the URP/Lit shader"
```

## 6.5 워크플로우 4: 에셋 생성 파이프라인

### AI Generators 활용

```
프롬프트 → AI가 필요한 에셋 분석 → Generator 실행 → 에셋 임포트
```

### 예시: 레벨 디자인

```
"RPG 게임의 숲 레벨을 만들어줘. 
필요한 것:
- 바닥 텍스처 (흙/잔디)
- 나무 프리팹 3종
- 바위 프리팹 2종
- 하늘 큐브맵 (숲 느낌)
- 주변 음향 효과"
```

### 생성된 에셋 관리

- 모든 AI 생성 에셋에 **AI 메타데이터** 자동 태깅
- EXIF: `ai-generated: true`, `tool: unity-ai`, `timestamp: ...`
- 앱스토어 제출 시 AI 생성 콘텐츠 **공개 의무** 대비

## 6.6 워크플로우 5: 팀 협업

### AI Assistant 공유

- `.aiconfig` 파일로 설정 공유
- 팀 공통 Skills 패키지화
- Git으로 AI 설정 관리

### MCP Server 팀 설정

```json
// 공통 MCP 설정 (팀 레포지토리)
{
  "mcpServers": {
    "unity": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"],
      "env": {
        "UNITY_MCP_HOST": "127.0.0.1",
        "UNITY_MCP_PORT": "51279"
      }
    }
  }
}
```

### 코드 리뷰 with AI

```
"최근 5개의 커밋을 분석해서 잠재적 버그를 찾아줘"
→ Git diff 분석 → MCP로 씬 영향도 확인
```

## 6.7 워크플로우 6: 지속적 통합(CI)

### GitHub Actions + Unity MCP

```yaml
# .github/workflows/unity-mcp-test.yml
name: Unity MCP Test
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Unity Tests via MCP
        run: |
          npx unity-mcp-server execute_code \
            '{"code": "TestRunner.RunAllTests();"}'
```

## 6.8 Skills 조합 워크플로우

여러 Skill을 조합하여 복잡한 작업 수행:

```
1. "Analyze Project" → 프로젝트 구조 파악
2. "Create Script" → 필요한 스크립트 생성
3. "Modify Scene" → 씬에 오브젝트 배치
4. "Generate Material" → 비주얼 리소스 생성
```

### 커스텀 Skill 체인 예시

```yaml
# skill-chain.yaml
workflow: "Create Enemy Wave System"
steps:
  - skill: analyze_scene
    params: { target: "MainScene" }
  - skill: create_script
    params: { 
      name: "WaveSpawner",
      template: "spawner"
    }
  - skill: modify_scene
    params: {
      action: "add_component",
      target: "GameManager",
      component: "WaveSpawner"
    }
  - skill: create_prefab
    params: {
      from: "Enemy",
      name: "EnemyVariants"
    }
```
