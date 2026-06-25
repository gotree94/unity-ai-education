# 02. 설치 및 환경 설정

## 2.1 사전 요구 사항 확인

```powershell
# Unity 버전 확인
Unity --version  # 또는 Unity Hub에서 확인
```

- Unity **6.0.0 이상** 필요 (권장: 6.3+)
- Unity Cloud 계정 필요
- 인터넷 연결 필수

## 2.2 설치 단계

### Step 1: AI Assistant 패키지 설치

1. Unity Editor 실행
2. `Window > Package Manager` 열기
3. **Add package by technical name** 선택
4. 패키지명 입력:

```
com.unity.ai.assistant
```

5. **Add** 클릭하여 설치

또는 **AI Button** 방식:
- 에디터 툴바의 **AI 버튼** 클릭
- 자동으로 패키지 설치 안내

### Step 2: 이용 약관 동의

1. `Window > AI > Assistant` 열기
2. 화면 안내에 따라 이용 약관 동의

### Step 3: 크레딧 및 구독 설정

| 사용자 유형 | 방법 |
|------------|------|
| Personal/Student | **Start free trial** 클릭 → 14일 무료 평가판 시작 (신용카드 필요) |
| Pro/Enterprise | 조직에서 자동 할당 → Cloud Dashboard에서 확인 |

### Step 4: 조직 설정 (Pro/Enterprise)

1. [Unity Cloud Dashboard](https://cloud.unity.com/) 접속
2. **Organization Owner 또는 Manager** 권한 필요
3. **AI > Assistant and Generators** 활성화

### Step 5: Unity Cloud 연결 확인

1. `Window > Unity Cloud` 열기
2. 프로젝트가 Unity Cloud에 연결되어 있는지 확인
3. 연결 안 되어 있으면 **Link** 버튼 클릭

## 2.3 설치 확인

### Assistant 실행 확인

```
Window > AI > Assistant
```

Assistant 창이 열리고 채팅 입력이 가능하면 설치 완료.

### 크레딧 확인

Assistant 창 하단에 남은 크레딧 표시 확인.

## 2.4 MCP Server 추가 설정

MCP Server는 AI Assistant 패키지에 **포함**되어 있습니다.

### MCP Server 활성화

1. `Edit > Project Settings` 열기
2. **AI > MCP Server** 탭 선택
3. **Enable MCP Server** 체크

### MCP Port 설정 (기본값)

| 설정 | 기본값 |
|------|--------|
| Host | 127.0.0.1 (localhost) |
| Port | 51279 |
| Transport | stdio |

## 2.5 환경 변수 (선택 사항)

```powershell
# MCP Server 환경 변수 설정 예시
$env:UNITY_MCP_HOST = "127.0.0.1"
$env:UNITY_MCP_PORT = "51279"
$env:UNITY_MCP_TIMEOUT = "60"
```

## 2.6 크레딧 소모 확인

Assistant 사용 시 크레딧이 소모됩니다. 예상 크레딧은 각 작업 전에 표시됩니다.

| 작업 유형 | 예상 크레딧 |
|----------|------------|
| C# 스크립트 생성 | ~20-50 |
| 씬 분석 | ~5-10 |
| 에셋 생성 | ~30-100 |
| Plan Mode 실행 | ~10-30 |

> **참고**: MCP Server를 통한 IDE-Editor 연결 시 크레딧이 **소모되지 않습니다**.

## 2.7 설치 후 첫 실행 체크리스트

- [ ] Unity 6.0+ 설치 확인
- [ ] AI Assistant 패키지 설치
- [ ] 이용 약관 동의
- [ ] Unity Cloud 연결
- [ ] 크레딧 할당/평가판 시작
- [ ] Assistant 창 열림 확인
- [ ] MCP Server 활성화 (선택)
