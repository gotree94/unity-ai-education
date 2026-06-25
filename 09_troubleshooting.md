# 09. 문제 해결 가이드

## 9.1 설치 관련 문제

### 패키지 설치 실패

| 증상 | 원인 | 해결 방법 |
|------|------|----------|
| "Package not found" | 패키지명 오타 | 정확한 이름 확인: `com.unity.ai.assistant` |
| 버전 호환성 오류 | Unity 버전 미달 | Unity 6.0+로 업데이트 |
| 네트워크 오류 | 인터넷 연결 | 방화벽/프록시 확인, VPN 비활성화 |
| "Already installed" | 중복 설치 | Package Manager에서 확인 후 재시도 |

### 크레딧 관련

| 증상 | 원인 | 해결 방법 |
|------|------|----------|
| "No credits available" | 크레딧 소진 | 추가 크레딧 구매 또는 평가판 시작 |
| "Subscription not found" | 구독 연결 안 됨 | Unity Cloud Dashboard 확인 |
| 크레딧이 줄지 않음 | 무료 작업 | MCP Server 작업은 크레딧 소모 없음 |

## 9.2 Assistant 관련 문제

### Assistant가 응답하지 않음

```
1. Window > AI > Assistant 재시작
2. Unity Editor 재시작
3. AI Assistant 패키지 재설치
4. Unity Cloud 연결 상태 확인
5. 인터넷 연결 확인
```

### 응답이 부정확함

| 문제 | 해결책 |
|------|--------|
| 컨텍스트를 이해 못 함 | 더 구체적인 프롬프트 제공 |
| 잘못된 코드 생성 | 생성된 코드를 설명과 함께 다시 요청 |
| 오래된 정보 사용 | Assistant에게 "현재 프로젝트 설정을 다시 읽어줘" |
| Plan Mode 무시 | Plan Mode가 켜져 있는지 확인 |

### Plan Mode가 작동하지 않음

```
1. Assistant 창에서 Plan Mode 토글 확인
2. Plan Mode는 복잡한 작업에만 활성화됨 (단순 질문은 Plan Mode 없음)
3. 에디터 재시작 후 재시도
```

## 9.3 MCP Server 문제

### 연결 실패

```
증상: "Connection refused" 또는 "Unable to connect to Unity"
```

**확인 사항:**
1. Unity Editor가 실행 중인가?
2. MCP Server가 활성화되어 있는가? (`Project Settings > AI > MCP Server`)
3. 포트 충돌이 있는가? (기본 51279)
4. 방화벽이 포트를 차단하고 있는가?
5. 같은 포트를 사용 중인 다른 프로세스가 있는가?

```powershell
# 포트 사용 확인
netstat -ano | Select-String "51279"

# 프로세스 종료
Stop-Process -Id (Get-Process -Id 
    (netstat -ano | Select-String "51279" 
    | ForEach-Object { $_ -split '\s+' | Select-Object -Last 1 }))
```

### MCP Tool 실행 오류

```json
// MCP Tool 오류 응답 예시
{
  "error": {
    "code": -32603,
    "message": "Internal error",
    "data": {
      "tool": "execute_code",
      "error": "NullReferenceException: ..."
    }
  }
}
```

**해결:**
1. Tool 이름/매개변수 확인
2. Unity 콘솔에서 상세 에러 확인
3. `execute_code` 사용 시 C# 문법 검증

### stdio 모드에서 실행 안 됨

```powershell
# npx/npm이 설치되어 있는지 확인
npx --version

# MCP 서버 직접 실행 테스트
npx -y unity-mcp-server --help
```

## 9.4 AI Gateway 문제

| 증상 | 원인 | 해결 |
|------|------|------|
| "Invalid API key" | API Key 오류 | Dashboard에서 재발급 |
| "Rate limit exceeded" | API 호출 제한 초과 | 잠시 대기 후 재시도 |
| "Model not available" | 모델명 오류 | 올바른 모델 ID 확인 |
| 느린 응답 시간 | 네트워크 지연 | 다른 모델로 전환 |

## 9.5 크레딧/청구 문제

### "Unexpected credit consumption"

```powershell
# 크레딧 사용 내역 확인
# Unity Cloud Dashboard > AI > Usage
```

**원인:**
- Plan Mode에서 승인한 작업이 예상보다 복잡했음
- 동일한 Assistant 세션에서 여러 번 수정 요청
- Generators 작업 (에셋 생성은 크레딧 소모 큼)

## 9.6 Generators 문제

| 증상 | 원인 | 해결 |
|------|------|------|
| 생성 실패 | 크레딧 부족 | 크레딧 추가 |
| 품질 저하 | 프롬프트 부족 | 더 구체적인 설명 추가 |
| 에셋 누락 | 생성 중단 | 재시도 |
| 잘못된 포맷 | Generator 제약 | 지원 포맷 확인 |

## 9.7 Skills 문제

```csharp
// Skill 디버깅 로그 추가
[Skill("DebugEnabledSkill")]
public class DebugEnabledSkill : ISkill
{
    public async Task<SkillResult> ExecuteAsync(
        SkillContext context, CancellationToken ct)
    {
        Debug.Log($"[Skill] Starting {Name}");
        
        try
        {
            Debug.Log("[Skill] Getting scene context...");
            var sceneContext = await context.GetSceneContextAsync();
            Debug.Log($"[Skill] Scene: {sceneContext.SceneName}");
            
            // ... 작업 수행 ...
            
            Debug.Log("[Skill] Completed successfully");
            return SkillResult.Success("완료");
        }
        catch (Exception ex)
        {
            Debug.LogError($"[Skill] Failed: {ex}");
            return SkillResult.Failure(ex.Message);
        }
    }
}
```

## 9.8 자주 묻는 오류 메시지

| 오류 메시지 | 의미 | 조치 |
|------------|------|------|
| "AI Assistant package not found" | 패키지 미설치 | `com.unity.ai.assistant` 설치 |
| "Failed to connect to Unity Cloud" | 클라우드 연결 실패 | 네트워크 확인, Cloud 재연결 |
| "Operation not allowed in Plan Mode" | Plan Mode 제한 | Plan Mode 해제 후 재시도 |
| "Skill execution timeout" | Skill 실행 시간 초과 | Skill 최적화 또는 단순화 |
| "MCP Tool not found" | Tool 이름 오타 | 올바른 Tool 이름 확인 |

## 9.9 로그 및 진단

```csharp
// MCP 진단 정보 출력
[MenuItem("AI/Diagnostics")]
static void ShowAIDiagnostics()
{
    Debug.Log("=== Unity AI Diagnostics ===");
    Debug.Log($"Unity Version: {Application.unityVersion}");
    Debug.Log($"AI Package: {GetAIPackageVersion()}");
    Debug.Log($"MCP Server: {IsMCPServerEnabled()}");
    Debug.Log($"Cloud Connected: {IsCloudConnected()}");
    Debug.Log($"Credits Remaining: {GetRemainingCredits()}");
}
```

**로그 파일 위치:**
- 에디터 로그: `%USERPROFILE%\AppData\Local\Unity\Editor\Editor.log`
- AI Assistant 로그: Package Manager 콘솔 출력 확인
- MCP Server 로그: Unity Console 창

## 9.10 지원 채널

| 채널 | 대상 | 응답 속도 |
|------|------|----------|
| Unity Discussions | 커뮤니티 | 수시간~수일 |
| Unity Support | 유료 사용자 | 1-2일 |
| Unity AI 피드백 | 베타 피드백 | 정기 업데이트 |
