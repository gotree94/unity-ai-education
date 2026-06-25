# 01. Unity AI Open Beta 개념 이해

## 1.1 Unity AI란?

**Unity AI**는 Unity 6+ 에디터에 내장된 **AI 도구 모음(Suite)** 입니다. 2026년 5월 4일 오픈 베타로 공개되었으며, 게임 개발자의 생산성을 획기적으로 높이는 것을 목표로 합니다.

### Muse와의 차이점

| 구분 | Unity Muse (구버전) | Unity AI (신버전) |
|------|-------------------|-------------------|
| 모델 | Unity 자체 모델 | Gemini 등 외부 Frontier Model |
| 위치 | 별도 창 | 에디터 내장 (프로젝트 컨텍스트 인지) |
| 방식 | 단순 채팅/자동완성 | **Agentic Workflow** (계획-실행-롤백) |
| 컨텍스트 | 제한적 | 씬, GameObject, 패키지, 플랫폼 등 전체 인지 |

## 1.2 핵심 구성 요소

### ① AI Assistant (인-프로젝트 에이전트)

- Unity **에디터 내부**에서 실행되는 **Agentic Assistant**
- 프로젝트 컨텍스트를 이해: 현재 씬, GameObject, 컴포넌트, 패키지, 빌드 설정
- **Plan Mode**: 실행 전 수행할 작업을 미리 검토 가능
- **Skills**: 기능 확장 시스템
- **Rollback**: 변경 사항 즉시 되돌리기 가능
- C# 스크립트 생성, 씬 편집, 에셋 생성 등 수행

### ② AI Gateway (AI 게이트웨이)

- 에디터를 떠나지 않고 **외부 AI 모델/서비스** 연결
- Claude, GPT, Gemini 등을 Unity 내에서 직접 사용
- AI Assistant와 동일한 프로젝트 컨텍스트 활용
- 보안/안정성 최적화된 연결

### ③ MCP Server (MCP 서버)

- **Model Context Protocol** 기반 오픈 표준 연결
- Claude Code, Cursor, Windsurf, VS Code Copilot 등의 IDE에서 Unity Editor 제어
- 실시간 씬 정보 조회, 코드 실행, 에디터 액션 수행
- AI 크레딧 **소모 없음** (무료)

### ④ Generators (생성기)

- 플레이스홀더 머티리얼, 사운드, 큐브맵, 2D/3D 에셋 생성
- 생성된 에셋에는 EXIF 메타데이터로 **AI 생성 태그** 자동 부착

### ⑤ Skills (스킬)

- AI Assistant의 기능을 확장하는 플러그인 시스템
- Plan Mode에서 Skill 실행 계획 수립 가능
- Unity 패키지 형태로 배포

## 1.3 Agentic Workflow 이해하기

Unity AI의 핵심은 **단순 채팅**이 아니라 **Agentic Workflow**입니다.

1. **요청**: 사용자가 자연어로 작업 요청
2. **계획**: AI가 수행할 작업 단계를 수립하고 Plan Mode로 검토 제공
3. **실행**: 계획된 각 단계를 순차적으로 실행
4. **검증**: 실행 결과 확인 및 필요시 수정
5. **롤백**: 문제 발생 시 변경 사항을 이전 상태로 복구

```
사용자: "씬에 빨간 큐브를 만들고 중력을 적용해줘"
    │
    ▼
AI 계획: 1. GameObject > 3D Object > Cube 생성
         2. 위치 (0, 5, 0) 설정
         3. 머티리얼 생성 > 색상 빨강 설정
         4. Rigidbody 컴포넌트 추가
         5. Plan Mode에서 사용자 승인 대기
    │
    ▼
사용자 승인 → AI 실행 → 결과 확인
```

## 1.4 MCP(Model Context Protocol) 개념

**MCP**는 Anthropic이 만든 **오픈 표준 프로토콜**로, AI 에이전트가 외부 도구와 통신하는 방식을 표준화합니다.

- 2024년 11월: Anthropic이 최초 발표
- 2025년 12월: Linux Foundation의 Agentic AI Foundation으로 이관
- 2026년 기준: 97M+ SDK 다운로드, 13,000+ MCP 서버

### MCP의 핵심 요소

| 요소 | 설명 |
|------|------|
| **Tools** | AI가 호출할 수 있는 함수 (씬 조회, 코드 실행 등) |
| **Resources** | AI가 읽을 수 있는 데이터 (파일, 로그 등) |
| **Prompts** | 미리 정의된 프롬프트 템플릿 |
| **Transport** | stdio (로컬) 또는 Streamable HTTP (원격) |

## 1.5 크레딧 시스템

| 사용자 유형 | 무료 평가판 | 정기 크레딧 |
|------------|------------|------------|
| Personal/Student | 14일 / 1,000 크레딧 | $10/월 / 1,000 크레딧 |
| Pro | 포함 | 2,000 크레딧/좌석/월 |
| Enterprise/Industry | 포함 | 3,000 크레딧/좌석/월 |

- MCP Server 사용 시 크레딧 **소모되지 않음**
- 추가 크레딧 번들 구매 가능

## 1.6 보안 및 프라이버시

- **기본값**: 프롬프트, 응답, 상호작용을 모델 학습에 **사용하지 않음**
- **옵트인**: Unity Dashboard에서 데이터 공유 선택 가능
- MCP Server는 로컬 IPC 아키텍처로 네트워크 노출 없음
- AI 생성 에셋은 EXIF 메타데이터 태깅 (앱스토어 규제 대응)

## 1.7 요구 사항

- Unity Editor **6.0 (Unity 6)** 이상 (권장: 6.3+)
- Unity Cloud에 프로젝트 연결
- 인터넷 연결
- Unity AI 이용 약관 동의
