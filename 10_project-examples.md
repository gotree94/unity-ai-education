# 10. 실전 프로젝트 예제

## 10.1 예제 1: 3D 점프 게임 제작

### 목표
Unity AI로 15분 만에 간단한 3D 점프 게임 프로토타입 제작

### 단계

**Step 1** — 기본 씬 생성

```
"3D 점프 게임 씬을 만들어줘.
- 바닥 (평면)
- 플레이어 (구체, 노란색)
- 점프대 3개 (빨간색 큐브)
- 도착 지점 (초록색 큐브)
Plan Mode로 검토 후 실행"
```

**Step 2** — 플레이어 스크립트

```
"플레이어 스크립트를 만들어줘:
- 스페이스바 누르면 점프
- 점프 높이는 5
- 점프대 위에 착지하면 2배로 점프
- 방향키로 좌우 이동
- 중력은 9.81"
```

**Step 3** — 게임 매니저

```
"게임 매니저 스크립트:
- 도착 지점에 닿으면 레벨 클리어
- UI에 'Level Complete!' 표시
- R 키로 리스폰
- 타이머 표시"
```

**Step 4** — UI

```
"게임 UI 만들어줘 (Canvas):
- 상단 좌측: 타이머
- 상단 우측: 시도 횟수
- 중앙: 게임 오버/클리어 텍스트
- 폰트: Arial, 흰색, 크기 36"
```

### 결과

| 항목 | 소요 시간 | 비고 |
|------|----------|------|
| 씬 구성 | ~2분 | Plan Mode 1회 승인 |
| 스크립트 3종 | ~5분 | 2회 수정 요청 |
| UI 구성 | ~3분 | 한 번에 완성 |
| 테스트 및 조정 | ~5분 | 점프 높이 조정 |
| **총 소요** | **~15분** | |

## 10.2 예제 2: 2D 슈팅 게임 — MCP Server 활용

### 목표
Claude Code + MCP Server로 Unity 프로젝트 제어

### 설정

```json
// .vscode/mcp.json
{
  "mcpServers": {
    "unity": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"]
    }
  }
}
```

### Claude Code 세션

```bash
# 1. 새 프로젝트 생성
"Create a new 2D project in Unity"

# 2. 플레이어 생성
"Create a player GameObject as a sprite 
 at position (0, -4, 0). 
 Add a PlayerMovement script with arrow key controls."

# 3. 총알 시스템
"Create a bullet prefab (small yellow circle).
 Add a Bullet script that moves upward at speed 10.
 Create a BulletSpawner script on the player 
 that shoots on spacebar press."

# 4. 적 시스템
"Create enemy spawner that spawns enemies 
 from top of screen every 2 seconds.
 Enemies move downward at speed 3.
 Add a simple box collider to enemies."

# 5. 충돌 처리
"Add collision logic: 
 bullets destroy enemies, 
 enemies destroy player on contact.
 Add score tracking."
```

### 결과

```bash
# Claude Code로 전체 플로우 실행
# 모든 씬 변경, 스크립트 생성, 컴포넌트 추가를 
# MCP를 통해 Unity Editor가 직접 수행
```

## 10.3 예제 3: 레벨 디자인 자동화

### 목표
AI를 활용한 절차적 레벨 디자인

### 프롬프트

```
"Roguelike 던전 레벨을 만들어줘:
- 5x5 그리드 기반
- 방 타입: 시작, 전투, 보상, 보스, 종료
- 각 방은 프리팹으로 관리
- 방 사이는 복도로 연결
- 랜덤 시드 기반 생성"
```

### AI 생성 결과

```csharp
// DungeonGenerator.cs (AI 생성)
using System.Collections.Generic;
using UnityEngine;

public class DungeonGenerator : MonoBehaviour
{
    [SerializeField] private int gridSize = 5;
    [SerializeField] private GameObject[] roomPrefabs;
    [SerializeField] private int randomSeed = 42;
    
    void Start()
    {
        GenerateDungeon();
    }
    
    void GenerateDungeon()
    {
        Random.InitState(randomSeed);
        
        for (int x = 0; x < gridSize; x++)
        {
            for (int y = 0; y < gridSize; y++)
            {
                int roomIndex = GetRoomTypeForPosition(x, y);
                Vector3 position = new Vector3(x * 10, 0, y * 10);
                Instantiate(roomPrefabs[roomIndex], 
                    position, Quaternion.identity, transform);
            }
        }
        
        ConnectRooms();
    }
    
    int GetRoomTypeForPosition(int x, int y)
    {
        if (x == 0 && y == 0) return 0;        // Start
        if (x == gridSize-1 && y == gridSize-1) return 4; // Boss
        return Random.Range(1, 4);  // Combat/Reward/Empty
    }
    
    void ConnectRooms()
    {
        // 복도 생성 로직
        Debug.Log("Rooms connected with corridors");
    }
}
```

## 10.4 예제 4: AI Gateway 멀티모델 활용

### 목표
작업별로 최적의 AI 모델 사용

### 설정

```json
{
  "aiGateway": {
    "providers": [
      {
        "name": "google",
        "model": "gemini-1.5-pro",
        "default": true
      },
      {
        "name": "anthropic",
        "apiKey": "sk-ant-xxx",
        "model": "claude-sonnet-4-20260506"
      },
      {
        "name": "openai",
        "apiKey": "sk-xxx",
        "model": "gpt-4.1"
      }
    ]
  }
}
```

### 작업별 모델 선택

| 작업 | 추천 모델 | 이유 |
|------|----------|------|
| C# 코드 생성 | Claude Sonnet 4 | 코드 품질 우수 |
| 씬 분석 | Gemini 1.5 Pro | 긴 컨텍스트 처리 |
| UI 디자인 제안 | GPT-4.1 | 창의적 디자인 |
| 버그 수정 | Claude Sonnet 4 | 정확한 분석 |
| 에셋 생성 | Gemini 1.5 Pro | 멀티모달 이해 |

## 10.5 예제 5: 커스텀 Skill 제작 — 자동 리팩토링

### Skill: Code Refactoring Tool

```csharp
[Skill("AutoRefactor", 
    Description = "선택된 스크립트를 자동 리팩토링",
    Version = "1.0.0")]
public class AutoRefactorSkill : ISkill
{
    public async Task<SkillResult> ExecuteAsync(
        SkillContext context, CancellationToken ct)
    {
        var selectedObjects = context.GetSelectedObjects();
        int refactoredCount = 0;
        var results = new List<string>();
        
        foreach (var obj in selectedObjects)
        {
            ct.ThrowIfCancellationRequested();
            
            var monoBehaviour = obj.GetComponent<MonoBehaviour>();
            if (monoBehaviour == null) continue;
            
            var script = MonoScript.FromMonoBehaviour(monoBehaviour);
            var path = AssetDatabase.GetAssetPath(script);
            var code = File.ReadAllText(path);
            
            // 리팩토링 규칙 적용
            code = RemoveUnusedUsings(code);
            code = ExtractMagicNumbers(code);
            code = AddNullChecks(code);
            code = SimplifyLinq(code);
            
            // 파일 저장
            File.WriteAllText(path, code);
            AssetDatabase.ImportAsset(path);
            
            refactoredCount++;
            results.Add($"{script.name}: 리팩토링 완료");
        }
        
        return SkillResult.Success(
            $"{refactoredCount}개 스크립트 리팩토링 완료\n" +
            string.Join("\n", results));
    }
    
    string RemoveUnusedUsings(string code) { /* ... */ return code; }
    string ExtractMagicNumbers(string code) { /* ... */ return code; }
    string AddNullChecks(string code) { /* ... */ return code; }
    string SimplifyLinq(string code) { /* ... */ return code; }
}
```

## 10.6 예제 6: CI/CD 통합

### GitHub Actions + Unity MCP

```yaml
name: Unity AI Build Pipeline
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: unity-2022.3
    steps:
      - uses: actions/checkout@v4
      
      - name: MCP Build
        run: |
          npx unity-mcp-server execute_code '{
            "code": "BuildPipeline.BuildPlayer(
              EditorBuildSettings.scenes,
              \"Builds/Game.exe\",
              BuildTarget.StandaloneWindows64,
              BuildOptions.None
            );"
          }'
      
      - name: AI Code Review
        run: |
          npx unity-mcp-server execute_code '{
            "code": "Debug.Log(
              AIAssistant.RunSkill(
                \"AnalyzeProject\",
                new { checkPerformance = true }
              )
            );"
          }'
```

## 10.7 프로젝트 템플릿

### Unity AI 프로젝트 시작 템플릿

```
MyGame/
├── Assets/
│   ├── AI_Generated/         # AI 생성 에셋
│   │   ├── Scripts/
│   │   ├── Materials/
│   │   └── Prefabs/
│   ├── Manual/               # 수동 제작 에셋
│   │   ├── Scripts/
│   │   ├── Art/
│   │   └── Audio/
│   └── AI_Config/            # AI 설정
│       ├── .ai-rules.yaml    # AI 동작 규칙
│       └── skills/           # 커스텀 Skills
├── ProjectSettings/
│   └── AI/
│       └── MCPConfig.asset   # MCP 설정
└── .vscode/
    └── mcp.json              # VS Code MCP 설정
```
