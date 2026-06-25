# 07. 고급: 커스텀 도구 및 Skills 개발

## 7.1 Skills 시스템 이해

**Skills**는 Unity AI Assistant의 기능을 확장하는 플러그인 시스템입니다.

### Skill 구조

```
MySkill/
├── Editor/
│   ├── MySkill.cs          # Skill 정의
│   ├── MySkillEditor.cs    # 에디터 UI (선택)
│   └── MySkillSettings.cs  # 설정 (선택)
├── Runtime/
│   └── MySkillRuntime.cs   # 런타임 로직 (선택)
└── package.json            # 패키지 매니페스트
```

## 7.2 커스텀 Skill 만들기

### 기본 Skill 템플릿

```csharp
using UnityEditor;
using UnityEngine;
using Unity.AI.Assistant;
using Unity.AI.Assistant.Skills;

[Skill("MyCustomSkill", 
    Description = "내 커스텀 스킬 설명",
    Version = "1.0.0")]
public class MyCustomSkill : ISkill
{
    public string Name => "MyCustomSkill";
    public string Description => "원하는 작업을 수행하는 커스텀 스킬";
    
    // Skill이 실행 가능한 조건 정의
    public bool CanExecute(SkillContext context)
    {
        return true; // 항상 실행 가능
    }
    
    // Skill 실행 로직
    public async Task<SkillResult> ExecuteAsync(
        SkillContext context, 
        CancellationToken cancellationToken)
    {
        try
        {
            // 1. 현재 씬 컨텍스트 가져오기
            var sceneContext = await context.GetSceneContextAsync();
            
            // 2. 작업 수행
            var selectedObjects = Selection.gameObjects;
            int modifiedCount = 0;
            
            foreach (var obj in selectedObjects)
            {
                // 작업 수행
                Undo.RecordObject(obj, "My Custom Skill");
                obj.name = $"Modified_{obj.name}";
                modifiedCount++;
                
                cancellationToken.ThrowIfCancellationRequested();
            }
            
            // 3. 결과 반환
            return SkillResult.Success(
                $"{modifiedCount}개 오브젝트 수정 완료");
        }
        catch (OperationCanceledException)
        {
            return SkillResult.Cancelled("사용자가 취소함");
        }
        catch (System.Exception ex)
        {
            return SkillResult.Failure($"오류 발생: {ex.Message}");
        }
    }
}
```

### Skill Context API

```csharp
// SkillContext가 제공하는 주요 기능
public class SkillContext
{
    // 씬 정보
    Task<SceneContext> GetSceneContextAsync();
    
    // 선택된 오브젝트
    GameObject[] GetSelectedObjects();
    
    // 프로젝트 정보
    Task<ProjectContext> GetProjectContextAsync();
    
    // 콘솔 로그
    Task<ConsoleLog[]> GetConsoleLogsAsync();
    
    // 사용자 입력 요청
    Task<string> RequestUserInputAsync(string prompt);
    
    // Plan Mode에 제안할 액션
    void SuggestAction(string description, Action execute);
}
```

## 7.3 Skill 패키징 및 배포

### package.json

```json
{
  "name": "com.yourcompany.myskill",
  "version": "1.0.0",
  "displayName": "My Custom Skill",
  "description": "내 커스텀 Unity AI Skill",
  "unity": "6000.0",
  "keywords": ["unity", "ai", "skill"],
  "category": "AI Skills",
  "dependencies": {
    "com.unity.ai.assistant": "2.7.0"
  }
}
```

### 설치 방법

1. **Package Manager** > **Add package from git URL**
2. 또는 `Packages/manifest.json`에 추가:
```json
{
  "dependencies": {
    "com.yourcompany.myskill": "https://github.com/you/myskill.git"
  }
}
```

## 7.4 MCP 커스텀 Tool 개발

### C#에서 MCP Tool 등록

```csharp
using UnityEditor;
using UnityEngine;
using Unity.AI.Assistant.MCP;

[MCPTool("find_assets_by_type",
    Description = "특정 타입의 에셋 검색")]
public class FindAssetsByTypeTool : IMCPTool
{
    public string Name => "find_assets_by_type";
    public string Description => "타입별 에셋 검색";
    
    // 매개변수 정의
    public MCPParameter[] Parameters => new[]
    {
        new MCPParameter
        {
            Name = "type",
            Type = MCPParameterType.String,
            Description = "에셋 타입 (Material, Texture, Prefab 등)",
            Required = true
        },
        new MCPParameter
        {
            Name = "searchInFolders",
            Type = MCPParameterType.Array,
            Description = "검색 폴더 (선택)",
            Required = false
        }
    };
    
    // 실행 로직
    public async Task<MCPToolResult> ExecuteAsync(
        MCPToolContext context)
    {
        string type = context.GetParameter<string>("type");
        string[] folders = context.GetParameter<string[]>(
            "searchInFolders") ?? new[] { "Assets" };
        
        var guids = AssetDatabase.FindAssets(
            $"t:{type}", folders);
        
        var assets = guids.Select(guid => new
        {
            path = AssetDatabase.GUIDToAssetPath(guid),
            name = System.IO.Path.GetFileNameWithoutExtension(
                AssetDatabase.GUIDToAssetPath(guid))
        }).ToArray();
        
        return MCPToolResult.Success(assets);
    }
}
```

## 7.5 고급 MCP 활용

### 배치 작업 자동화

```bash
# MCP로 배치 파이프라인 실행
mcp-tools:
  - name: batch_optimize
    tools:
      - compress_textures:
          quality: 0.5
      - remove_unused_assets
      - analyze_performance
```

### 멀티 클라이언트 설정

```json
{
  "mcpServers": {
    "unity-main": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"],
      "env": {
        "UNITY_MCP_PORT": "51279"
      }
    },
    "unity-build": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"],
      "env": {
        "UNITY_MCP_PORT": "51280"
      }
    }
  }
}
```

## 7.6 AI Gateway 커스텀 공급자

```csharp
// 커스텀 AI 공급자 구현
[CustomAIGatewayProvider("my-custom-provider")]
public class MyCustomProvider : IAIGatewayProvider
{
    public string Name => "My Custom AI";
    
    public async Task<AIModelResponse> GenerateAsync(
        AIModelRequest request,
        CancellationToken cancellationToken)
    {
        // 자체 AI 모델 또는 서드파티 API 호출
        var httpClient = new HttpClient();
        httpClient.DefaultRequestHeaders.Add(
            "Authorization", $"Bearer {GetApiKey()}");
        
        var response = await httpClient.PostAsJsonAsync(
            "https://my-ai-api.example.com/generate",
            new {
                prompt = request.Prompt,
                context = request.Context
            },
            cancellationToken);
        
        var result = await response.Content
            .ReadFromJsonAsync<MyApiResponse>();
        
        return new AIModelResponse
        {
            Text = result.Text,
            Usage = new TokenUsage
            {
                PromptTokens = result.PromptTokens,
                CompletionTokens = result.CompletionTokens
            }
        };
    }
}
```

## 7.7 Skills 고급 패턴

### 체이닝 패턴

```csharp
[Skill("ComboSkill")]
public class ComboSkill : ISkill
{
    public async Task<SkillResult> ExecuteAsync(
        SkillContext context, CancellationToken ct)
    {
        // 여러 Skill을 조합
        var analyzeResult = await context.RunSkillAsync(
            "AnalyzeScene", ct);
        
        if (!analyzeResult.IsSuccess)
            return analyzeResult;
        
        var createResult = await context.RunSkillAsync(
            "CreateOptimizedLighting", 
            new { target = "MainScene" }, 
            ct);
        
        return createResult;
    }
}
```

### 조건부 실행 패턴

```csharp
public bool CanExecute(SkillContext context)
{
    var scene = context.GetSceneContextAsync().Result;
    return scene.GameObjectCount < 1000 
           && scene.HasComponent<Light>();
}
```

## 7.8 디버깅 및 프로파일링

```csharp
[Skill("DebugSkill")]
public class DebugSkill : ISkill
{
    public async Task<SkillResult> ExecuteAsync(
        SkillContext context, CancellationToken ct)
    {
        using var profileScope = new ProfilingScope("MySkill");
        
        var sw = Stopwatch.StartNew();
        
        // 작업 수행
        var result = await DoWorkAsync(context, ct);
        
        sw.Stop();
        Debug.Log($"MySkill completed in {sw.ElapsedMilliseconds}ms");
        
        return result;
    }
}
```
