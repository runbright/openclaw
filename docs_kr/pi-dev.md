---
title: "Pi 개발 워크플로우"
summary: "Pi 통합을 위한 개발자 워크플로우: 빌드, 테스트 및 라이브 검증 방법"
read_when:
  - Pi 통합 코드나 테스트 작업을 할 때
  - Pi 전용 린트, 타입 체크, 라이브 테스트 흐름을 실행할 때
---

# Pi 개발 워크플로우 (Pi Development Workflow)

이 가이드는 OpenClaw에서 Pi 에이전트 통합 작업을 할 때 권장되는 개발 흐름을 요약합니다.

## 타입 체크 및 린트

- 타입 체크 및 빌드: `pnpm build`
- 린트(Lint): `pnpm lint`
- 포맷 체크: `pnpm format`
- 푸시 전 전수 검사: `pnpm lint && pnpm build && pnpm test`

## Pi 전용 테스트 실행

Pi 관련 테스트 세트만 골라서 직접 실행하려면 Vitest를 사용합니다:

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

실제 API 제공자를 통한 실환경 테스트를 포함하려면:

```bash
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

## 수동 테스트 가이드

추천하는 수동 테스트 흐름:

- 게이트웨이를 개발 모드로 실행:
  - `pnpm gateway:dev`
- 직접 에이전트 호출:
  - `pnpm openclaw agent --message "안녕" --thinking low`
- 대화형 디버깅을 위해 TUI 사용:
  - `pnpm tui`

도구 호출 동작을 확인하려면 `read`나 `exec`가 필요한 질문을 던져서 도구 스트리밍과 데이터 처리 과정을 관찰하세요.

## 환경 초기화 (Reset)

상태 정보는 `~/.openclaw` (또는 `OPENCLAW_STATE_DIR` 환경 변수가 가리키는 곳)에 저장됩니다.

모든 것을 초기화하고 싶을 때:
- `openclaw.json`: 설정 파일
- `credentials/`: 인증 프로필 및 토큰
- `agents/<에이전트ID>/sessions/`: 대화 기록
- `agents/<에이전트ID>/sessions.json`: 세션 인덱스
- `workspace/`: 빈 워크스페이스를 원할 때 삭제

대화 기록만 지우고 싶다면 `agents/<에이전트ID>/sessions/`와 `sessions.json`만 삭제하세요. API 인증 정보를 유지하려면 `credentials/` 폴더는 남겨두어야 합니다.
---
