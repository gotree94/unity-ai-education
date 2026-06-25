# 08. 모범 사례 & 보안

## 8.1 프롬프트 엔지니어링 모범 사례

### 효과적인 프롬프트 구조

```
Bad:  "씬 수정해줘"

Good: "MainScene의 플레이어 오브젝트에 
       Rigidbody 컴포넌트를 추가하고, 
       질량을 2로 설정해줘. 
       Plan Mode로 검토 후 실행해줘"
```

### 원칙

| 원칙 | 설명 | 예시 |
|------|------|------|
| **구체적** | 무엇을, 어디에, 어떻게 | "빨간색" 대신 "RGB(255,0,0)" |
| **컨텍스트 제공** | 대상과 범위 명시 | "Hierarchy의 'Enemies' 폴더 아래" |
| **Plan Mode 활용** | 실행 전 검토 | 변경 예상치 못한 작업에 필수 |
| **단계 분할** | 복잡한 작업은 분할 | "1. 스크립트 생성 2. 컴포넌트 연결" |
| **되돌리기 가능** | Rollback 계획 | "변경 사항을 Git에 커밋해줘" |

## 8.2 프로젝트 관리

### 버전 관리와 AI

```bash
# AI 작업 전 브랜치 생성
git checkout -b feature/ai-generated-player-controller

# AI가 스크립트 생성
# ... (Assistant 사용)

# 변경 사항 검토 및 커밋
git diff
git commit -m "feat: AI-generated player controller"
```

### .gitignore에 추가

```gitignore
# AI 생성 임시 파일
*.ai_temp
*.ai_preview
```

## 8.3 크레딧 관리

### 크레딧 절약 전략

| 전략 | 설명 |
|------|------|
| **Plan Mode 우선** | 불필요한 실행 방지 |
| **간결한 프롬프트** | 긴 대화 = 더 많은 크레딧 |
| **MCP Server 활용** | 크레딧 소모 없는 작업은 MCP로 |
| **로컬 작업優先** | 네트워크가 필요 없는 작업 구분 |
| **크레딧 모니터링** | Assistant 하단 잔여 크레딧 확인 |

### 크레딧 소모 예측

| 작업 | 예상 소모량 |
|------|-----------|
| "간단한 C# 스크립트 생성" | ~20 크레딧 |
| "씬 분석 및 최적화 제안" | ~30 크레딧 |
| "3D 에셋 생성" | ~50-100 크레딧 |
| "전체 프로젝트 구조 분석" | ~40 크레딧 |

## 8.4 보안 모범 사례

### API Key 관리

```csharp
// BAD: 하드코딩
const string apiKey = "sk-ant-xxxxx";  // ❌

// GOOD: Secure Preferences 사용
string apiKey = CloudProjectSettings.apiKey;  // ✅

// GOOD: 환경 변수 사용
string apiKey = Environment.GetEnvironmentVariable(
    "UNITY_AI_API_KEY");  // ✅
```

### MCP Server 보안

| 설정 | 권장값 | 이유 |
|------|--------|------|
| Host | `127.0.0.1` | 외부 접근 차단 |
| Transport | `stdio` | HTTP보다 안전 |
| `execute_code` 사용 | 제한적 | 필요한 경우만 활성화 |
| 네트워크 방화벽 | 포트 차단 | 불필요한 노출 방지 |

### 코드 검증

```bash
# AI가 생성한 코드 검토 체크리스트
- [ ] 하드코딩된 값 없음
- [ ] 보안에 민감한 API Key 노출 없음
- [ ] 예외 처리 적절함
- [ ] 성능 이슈 없음
- [ ] 코드 스타일 일관성
```

## 8.5 AI 생성 에셋 관리

### 메타데이터 확인

```csharp
// AI 생성 에셋 메타데이터 읽기
var importer = AssetImporter
    .GetAtPath("Assets/Generated/Cube.mat");
    
if (importer.userData.Contains("ai-generated"))
{
    Debug.Log("This asset was AI-generated");
}
```

### 에셋 분류

```
Assets/
├── AI_Generated/          # AI로 생성된 모든 에셋
│   ├── Scripts/
│   ├── Materials/
│   ├── Prefabs/
│   └── Textures/
├── Manual/                # 수동으로 제작한 에셋
└── Mixed/                 # AI + 수동 편집
```

## 8.6 팀 협업 가이드라인

### 규칙 예시

```yaml
# .ai-rules.yaml (프로젝트 루트)
ai_assistant:
  plan_mode: required  # 모든 변경 전 Plan Mode 필수
  approval: required   # Senior 개발자 승인 필요
  banned_operations:
    - delete_scene
    - modify_build_settings
  review_required:
    - "*.cs"           # C# 스크립트는 항상 리뷰
    - "*.prefab"       # 프리팹 변경 리뷰
```

### AI 작업 로깅

```csharp
[InitializeOnLoadMethod]
static void LogAIActions()
{
    AIAssistant.OnActionExecuted += (action) =>
    {
        Debug.Log($"[AI] {action.Timestamp}: " +
                  $"{action.Description} by " +
                  $"{action.User}");
        
        // 로그 파일 저장
        File.AppendAllText(
            "AI_Actions.log",
            $"{action.Timestamp}\t{action.User}\t" +
            $"{action.Description}\t{action.Status}\n");
    };
}
```

## 8.7 성능 고려사항

| 고려사항 | 영향 | 권장 |
|----------|------|------|
| 대규모 씬 분석 | 크레딧, 시간 소모 | 필요한 부분만 선택적 분석 |
| Plan Mode | 실행 지연 | 중요 변경 시만 사용 |
| MCP 빈번 호출 | 에디터 성능 저하 | 1초 간격 호출 제한 |
| 동시 Assistant 요청 | 모델 과부하 | 순차적 요청 |

## 8.8 장애 조치 계획

1. **AI 생성 코드 검토**는 항상 사람이 수행
2. **Git 브랜치**로 AI 작업 분리
3. **자동 백업** 활성화
4. **롤백 절차** 숙지 (Undo History 사용)
5. 중요 작업 전 **수동 저장**
