# 04. AI Gateway — 외부 AI 모델 연결

## 4.1 AI Gateway란?

**AI Gateway**는 Unity AI Assistant가 Unity 에디터를 떠나지 않고 **외부 AI 모델 및 서비스**에 연결할 수 있게 해주는 기능입니다.

### 용도

- Claude, GPT, Gemini 등 선호하는 AI 모델을 Unity에서 직접 사용
- Assistant와 동일한 프로젝트 컨텍스트를 외부 모델이 활용
- 기존 AI 구독(API Key)이 있다면 추가 비용 없이 사용 가능

## 4.2 아키텍처

```
┌─────────────────────────────────────────┐
│              Unity Editor               │
│  ┌─────────────────────────────────┐    │
│  │      AI Assistant               │    │
│  │  ┌───────────────────────────┐  │    │
│  │  │      AI Gateway           │  │    │
│  │  └──────────┬────────────────┘  │    │
│  └─────────────┼───────────────────┘    │
└────────────────┼────────────────────────┘
                 │
    ┌────────────┼────────────┐
    ▼            ▼            ▼
┌────────┐ ┌────────┐ ┌────────┐
│ Gemini │ │ Claude │ │  GPT   │
│ (기본)  │ │ (선택)  │ │ (선택)  │
└────────┘ └────────┘ └────────┘
```

## 4.3 AI Gateway 설정

### Step 1: Gateway 설정 열기

```
Window > AI > Assistant > 설정 아이콘 > AI Gateway
```

또는

```
Edit > Project Settings > AI > AI Gateway
```

### Step 2: 공급자 추가

지원 공급자:

| 공급자 | 연결 방식 | 비고 |
|--------|----------|------|
| Google Gemini | 기본 내장 | Unity AI 기본 모델 |
| Anthropic Claude | API Key | 별도 구독 필요 |
| OpenAI GPT | API Key | 별도 구독 필요 |
| 기타 OpenAI 호환 | API Key + Endpoint | 커스텀 |

### Step 3: API Key 설정

```json
{
  "aiGateway": {
    "providers": [
      {
        "name": "anthropic",
        "apiKey": "sk-ant-xxxxxxxxxxxx",
        "model": "claude-sonnet-4-20260506",
        "baseUrl": "https://api.anthropic.com"
      },
      {
        "name": "openai",
        "apiKey": "sk-xxxxxxxxxxxxxxxx",
        "model": "gpt-4.1",
        "baseUrl": "https://api.openai.com/v1"
      }
    ],
    "defaultProvider": "anthropic"
  }
}
```

> **주의**: API Key는 절대 소스 코드에 하드코딩하지 말고 환경 변수나 Unity의 Secure Preferences 사용.

## 4.4 Gateway 사용하기

### Assistant에서 모델 전환

Assistant 창 상단의 모델 드롭다운에서 연결된 공급자를 선택:

```
[ Gemini 1.5 Pro ▼ ]      ← 기본 모델
  Gemini 1.5 Pro
  Claude Sonnet 4
  GPT-4.1
```

### 프롬프트 예시 (Claude 사용)

```
"Claude로 전환해서 이 씬의 최적화 방안을 분석해줘"
```

## 4.5 AI Gateway의 장점

| 장점 | 설명 |
|------|------|
| **통합 컨텍스트** | 외부 모델도 Unity 프로젝트 전체 컨텍스트 활용 가능 |
| **유연성** | 작업별로 최적의 모델 선택 가능 |
| **비용 효율** | 기존 API 구독 활용 (이중 결제 불필요) |
| **보안** | Unity Cloud를 통한 안전한 키 관리 |

## 4.6 Rate Limiting 및 비용 관리

- 각 공급자별 Rate Limit 적용
- Assistant 사용량과 별도로 API Key 사용량은 해당 공급자에 직접 청구
- Unity Dashboard에서 Gateway 사용량 모니터링

## 4.7 커스텀 공급자 연결

OpenAI 호환 API를 제공하는 모든 서비스 연결 가능:

```json
{
  "name": "custom-provider",
  "apiKey": "sk-custom-key",
  "model": "custom-model-name",
  "baseUrl": "https://custom-api.example.com/v1"
}
```

## 4.8 문제 해결

| 증상 | 확인 사항 |
|------|----------|
| 연결 실패 | API Key 유효성, 네트워크 연결 |
| 느린 응답 | Rate Limit 도달, 모델 과부하 |
| 잘못된 응답 | 모델 전환, 프롬프트 구체화 |
| 크레딧 표시 안 됨 | Gateway 사용 시 Unity 크레딧 소모 없음 (별도 API 과금) |
