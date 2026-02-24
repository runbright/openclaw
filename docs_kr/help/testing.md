---
summary: "테스트 키트 안내: 유닛/E2E/라이브 테스트 슈트, Docker 실행기 및 테스트 범위 가이드"
read_when:
  - 로컬 환경이나 CI에서 테스트를 실행해야 할 때
  - 모델이나 제공업체의 버그 수정을 위한 테스트를 추가할 때
title: "테스트"
---

# 테스트 (Testing)

OpenClaw는 시스템의 안정성을 보장하기 위해 세 가지 수준의 테스트 슈트(Vitest 기반)와 Docker 실행 환경을 제공합니다.

## 테스트 종류 요약

| 테스트 슈트 | 명령어 | 설명 | 특징 |
| :--- | :--- | :--- | :--- |
| **단위 테스트 (Unit/Integration)** | `pnpm test` | 개별 함수나 모듈의 기능을 검증합니다. | 빠르고 CI에서 기본 실행됨 (API 키 불필요) |
| **E2E 테스트 (Smoke)** | `pnpm test:e2e` | 게이트웨이와 노드 간의 통신 등 전체 흐름을 검증합니다. | 실제 API 호출 없이 시뮬레이션으로 진행 |
| **라이브 테스트 (Live)** | `pnpm test:live` | **실제 API 키**를 사용하여 모델이 정상 작동하는지 확인합니다. | 실제 비용 발생, 네트워크 의존적 |

## 빠른 시작 가이드

- **코드 수정 후 기본 확인 (권장)**:
  ```bash
  pnpm build && pnpm check && pnpm test
  ```
- **전체 시나리오 테스트 (E2E)**:
  ```bash
  pnpm test:e2e
  ```
- **실제 모델 작동 확인 (라이브 - API 키 필요)**:
  ```bash
  pnpm test:live
  ```

## 라이브 테스트 상세 (Live Tests)

라이브 테스트는 실제 AI 제공업체(OpenAI, Anthropic, Google 등)의 API 변화나 작동 여부를 확인하는 데 사용됩니다.

- **모델 선택**: `OPENCLAW_LIVE_MODELS="openai/gpt-5.2"` 처럼 특정 모델만 골라서 테스트할 수 있습니다.
- **테스트 레이어**:
    - **Layer 1 (Direct)**: 모델이 질문에 답을 할 수 있는 상태인지 확인합니다.
    - **Layer 2 (Gateway)**: 세션 관리, 도구(Tools) 호출, 이미지 인식 등 전체 시스템 안에서 모델이 잘 작동하는지 확인합니다.

## Docker 테스트 (Docker Runners)

리눅스 환경에서의 작동을 로컬에서 미리 확인하고 싶을 때 유용합니다.

- `pnpm test:docker:live-models`: Docker 컨테이너 안에서 모델 테스트 실행
- `pnpm test:docker:onboard`: 초기 설정 마법사(Onboarding) 시뮬레이션

---
상세한 테스트 구현 방법은 `src/**/*.test.ts` 파일들을 참고하세요.
---
