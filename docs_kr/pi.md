---
title: "Pi 통합 아키텍처"
summary: "OpenClaw의 내장형 Pi 에이전트 통합 방식 및 세션 생명주기 아키텍처"
read_when:
  - OpenClaw 내부에서 Pi SDK가 어떻게 통합되었는지 이해하고 싶을 때
  - 에이전트 세션 생명주기, 도구 사용 또는 제공자 연결 방식을 수정할 때
---

# Pi 통합 아키텍처 (Pi Integration Architecture)

이 문서는 OpenClaw가 어떻게 [pi-coding-agent](https://github.com/badlogic/pi-mono) 및 관련 패키지들을 사용하여 AI 에이전트 기능을 구현하는지 설명합니다.

## 개요 (Overview)

OpenClaw는 Pi SDK를 사용하여 메시징 게이트웨이 아키텍처 내에 AI 코딩 에이전트를 내장(Embedded)합니다. Pi를 별도의 프로세스로 실행하거나 RPC 모드를 사용하는 대신, OpenClaw는 Pi의 `AgentSession`을 직접 가져와 인스턴스화합니다.

이러한 내장형 접근 방식의 장점은 다음과 같습니다:
- 세션 생명주기 및 이벤트 처리에 대한 완전한 제어
- 커스텀 도구 주입 (메시징, 샌드박스, 채널 전용 액션 등)
- 채널/컨텍스트별 시스템 프롬프트 맞춤 설정
- 제공자에 구애받지 않는 모델 전환 및 실패 시 복구(Failover)

## 패키지 의존성

OpenClaw는 다음과 같은 Pi 핵심 패키지들을 사용합니다:
- `pi-ai`: LLM 추상화 및 제공자 API
- `pi-agent-core`: 에이전트 루프, 도구 실행 로직
- `pi-coding-agent`: 고수준 SDK (세션 관리, 인증 저장소 등)

## 주요 통합 흐름 (Core Integration Flow)

### 1. 내장형 에이전트 실행
`pi-embedded-runner/run.ts`의 `runEmbeddedPiAgent()`가 주요 진입점입니다. 이 함수는 세션 ID, 워크스페이스 경로, 프롬프트 내용 등을 인자로 받아 실행됩니다.

### 2. 세션 생성
Pi SDK의 `createAgentSession()`을 사용하여 에이전트 세션을 구성합니다. 이때 OpenClaw가 정의한 도구 목록, 인증 저장소, 모델 레지스트리 등이 주입됩니다.

### 3. 이벤트 구독 (Event Subscription)
`subscribeEmbeddedPiSession()`은 세션 중에 발생하는 다양한 이벤트(메시지 업데이트, 도구 실행 시작/종료, 턴 종료 등)를 구독하고 적절한 콜백을 호출합니다.

## 도구 아키텍처 (Tool Architecture)

OpenClaw는 Pi의 기본 도구 세트를 확장하거나 대체하여 사용합니다:
1. **기본 도구**: Pi의 `codingTools` (읽기, 쓰기, 편집 등)
2. **OpenClaw 전용 도구**: 메시징, 브라우저 제어, 캔버스, 세션 관리 등
3. **채널 전용 도구**: Telegram, Discord, Slack, WhatsApp 등 각 플랫폼별 특화 기능
4. **정책 필터링**: 설정된 프로필이나 보안 정책에 따라 사용할 수 있는 도구를 걸러냅니다.

## 세션 관리 (Session Management)

- **세션 파일**: `.jsonl` 형식으로 저장되며 트리 구조를 가집니다.
- **기록 제한**: 채널 유형(DM vs 그룹)에 따라 대화 기록의 길이를 자동으로 조절합니다.
- **데이터 압축 (Compaction)**: 컨텍스트 창이 가득 차면 자동으로 대화 내용을 요약하거나 정리합니다.

## 제공자별 특이사항 처리
- **Anthropic (Claude)**: 프롬프트 인젝션 방어 및 특정 파라미터 호환성 보장
- **Google (Gemini)**: 턴 순서 교정 및 도구 스키마 최적화
- **OpenAI (GPT)**: `apply_patch` 도구 지원 및 모델별 특성 반영

---
상세한 코드 구현이나 기여 방법은 [영문 원본 문서](/pi)를 참고하시거나, 소스 코드의 `src/agents/` 디렉토리를 확인해 주시기 바랍니다.
---
