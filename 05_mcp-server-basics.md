# 05. MCP Server 기본 개념과 설정

## 5.1 MCP(Model Context Protocol) 개요

**MCP**는 AI 에이전트가 **외부 도구 및 데이터 소스**와 통신할 수 있는 **오픈 표준 프로토콜**입니다.

- 창시자: Anthropic (2024년 11월)
- 표준 관리: Linux Foundation Agentic AI Foundation (2025년 12월~)
- JSON-RPC 2.0 기반

### 왜 MCP인가?

| 상황 | MCP 없음 | MCP 있음 |
|------|---------|---------|
| 컨텍스트 | AI가 보는 건 사용자가 붙여넣은 코드뿐 | 실시간 씬 계층, GameObject 값, 콘솔 로그 등 접근 |
| 작업 | 수동으로 복사-붙여넣기 | AI가 직접 Unity Editor API 호출 |
| 반복성 | 매번 컨텍스트 재설명 | 지속적인 연결 유지 |

## 5.2 Unity MCP Server 아키텍처

```
┌─────────────────────┐     MCP Protocol      ┌─────────────────────┐
│   IDE / 터미널      │ ◄──────────────────►  │   Unity Editor      │
│                     │     (JSON-RPC 2.0)     │                     │
│  ┌───────────────┐  │                        │  ┌───────────────┐  │
│  │ Claude Code   │  │                        │  │ MCP Server    │  │
│  │ Cursor        │  │                        │  │ (내장)         │  │
│  │ Windsurf      │  │                        │  └───────────────┘  │
│  │ VS Code Copilot│  │                        │  ┌───────────────┐  │
│  └───────────────┘  │                        │  │ Unity Editor  │  │
│                     │                        │  │ API           │  │
└─────────────────────┘                        │  └───────────────┘  │
                                                └─────────────────────┘
```

### 통신 방식

| Transport | 용도 | 특징 |
|-----------|------|------|
| **stdio** | 기본 (로컬) | MCP 클라이언트가 서버 프로세스 실행, 가장 안전 |
| **Streamable HTTP** | 원격/고급 | 별도 HTTP 서버로 실행, LAN 접근 가능 |

## 5.3 Unity MCP Server 활성화

### 설치

AI Assistant 패키지(`com.unity.ai.assistant`)에 MCP Server가 포함되어 있습니다.

### 활성화

1. Unity Editor 실행
2. `Edit > Project Settings` 열기
3. **AI > MCP Server** 선택
4. **Enable MCP Server** 체크
5. 포트 확인 (기본: 51279)

## 5.4 IDE 클라이언트 설정

### Claude Code

```json
// claude.json 또는 .mcp.json
{
  "mcpServers": {
    "unity": {
      "command": "npx",
      "args": [
        "-y",
        "@anthropic/claude-code-mcp",
        "--unity"
      ]
    }
  }
}
```

### Cursor

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "unity": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"]
    }
  }
}
```

### VS Code (Copilot)

```json
// .vscode/mcp.json
{
  "mcpServers": {
    "unity": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"],
      "type": "stdio"
    }
  }
}
```

### Windsurf

```json
// .windsurf/mcp.json
{
  "mcpServers": {
    "unity": {
      "command": "npx",
      "args": ["-y", "unity-mcp-server"]
    }
  }
}
```

## 5.5 제공되는 MCP Tools

| 도구 | 설명 |
|------|------|
| `get_scene_hierarchy` | 현재 씬 계층 구조 조회 |
| `get_gameobject_info` | 특정 GameObject 정보 확인 |
| `find_gameobjects` | 조건에 맞는 GameObject 검색 |
| `execute_code` | Unity Editor에서 C# 코드 실행 |
| `create_gameobject` | 새 GameObject 생성 |
| `modify_component` | 컴포넌트 속성 변경 |
| `get_console_logs` | 콘솔 로그 조회 |
| `set_playmode` | 재생 모드 제어 (Play/Stop) |
| `manage_asset` | 에셋 가져오기/생성/삭제 |
| `batch_execute` | 여러 명령을 묶어서 실행 |

## 5.6 MCP Server 기본 사용 예시

### Claude Code에서 Unity 씬 조회

```bash
# Claude Code 터미널에서
# "List all GameObjects in my Unity scene"
```

AI가 MCP를 통해 `get_scene_hierarchy` 호출 → 결과 반환.

### Cursor에서 코드 수정

```bash
# Cursor 채팅에서
# "Find all enemies in the scene and add a health bar component to them"
```

AI가 `find_gameobjects("Enemy")` → `execute_code`로 컴포넌트 추가.

### 자동화 워크플로우

```bash
# "Fix all the warnings in the console"
```

1. `get_console_logs` → 경고 수집
2. `execute_code` → 각 경고 수정
3. `get_console_logs` → 수정 확인

## 5.7 MCP Server를 통한 자동화

### CI/CD 파이프라인 통합

```bash
# 배치 스크립트 예시
unity-mcp --batch --execute "BuildUtils.BuildGame()"
```

### 에디터 확장

```python
# Python 스크립트에서 Unity MCP 호출
import json, subprocess

def unity_mcp_call(tool_name, args):
    cmd = ["npx", "-y", "unity-mcp-server", tool_name, json.dumps(args)]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return json.loads(result.stdout)

# Unity 씬 정보 조회
scene = unity_mcp_call("get_scene_hierarchy", {})
print(scene)
```

## 5.8 보안 고려사항

| 위험 | 설명 | 대책 |
|------|------|------|
| 코드 실행 | `execute_code`로 임의 C# 실행 가능 | 신뢰할 수 있는 AI 에이전트만 사용 |
| 프롬프트 인젝션 | 악의적 프롬프트로 코드 실행 유도 | 입력 검증, Plan Mode 활용 |
| 네트워크 노출 | HTTP 모드 사용 시 외부 접근 가능 | localhost(127.0.0.1)만 바인딩 |

### 권장 보안 설정

1. **stdio 모드** 사용 (기본값, 가장 안전)
2. HTTP 모드 필요 시 **127.0.0.1**로 제한
3. 민감한 프로젝트는 MCP Server 비활성화
4. AI Agent의 변경 사항은 Git으로 추적

## 5.9 Unity 공식 MCP vs 커뮤니티 MCP

| 항목 | Unity 공식 MCP | Unity-MCP (IvanMurzak) | mcp-unity (CoderGamester) |
|------|---------------|----------------------|--------------------------|
| 패키지 | `com.unity.ai.assistant` 내장 | GitHub/Asset Store | GitHub |
| 요구사항 | Unity 6+, Unity Cloud | Unity 2021.3+ | Unity 2022.3+, Node.js |
| 특징 | 공식 지원, Plan Mode 연동 | 임의 C# 메서드→MCP Tool | WebSocket 기반 |
| 크레딧 | **필요 없음** | 불필요 | 불필요 |
| 라이선스 | Unity EULA | MIT | MIT |
