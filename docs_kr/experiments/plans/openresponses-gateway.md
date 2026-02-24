---
summary: "계획: OpenResponses /v1/responses 엔드포인트 추가 및 chat completions 깔끔한 폐기"
read_when:
  - "`/v1/responses` Gateway 지원 설계 또는 구현 시"
  - Chat Completions 호환성에서의 마이그레이션 계획 시
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
title: "OpenResponses Gateway 계획"
---

# OpenResponses Gateway 통합 계획

## 컨텍스트

OpenClaw Gateway는 현재 `/v1/chat/completions`에서 최소한의 OpenAI 호환 Chat Completions 엔드포인트를
노출합니다 ([OpenAI Chat Completions](/gateway/openai-http-api) 참조).

Open Responses는 OpenAI Responses API를 기반으로 한 오픈 추론 표준입니다. 에이전트 워크플로우를 위해
설계되었으며 항목 기반 입력과 시맨틱 스트리밍 이벤트를 사용합니다. OpenResponses 스펙은
`/v1/chat/completions`가 아닌 `/v1/responses`를 정의합니다.

## 목표

- OpenResponses 시맨틱을 준수하는 `/v1/responses` 엔드포인트 추가.
- Chat Completions를 비활성화하기 쉽고 최종적으로 제거할 수 있는 호환성 레이어로 유지.
- 분리된 재사용 가능한 스키마로 유효성 검사 및 파싱 표준화.

## 비목표

- 첫 번째 패스에서 전체 OpenResponses 기능 패리티 (이미지, 파일, 호스팅된 도구).
- 내부 에이전트 실행 로직 또는 도구 오케스트레이션 대체.
- 첫 번째 단계에서 기존 `/v1/chat/completions` 동작 변경.

## 리서치 요약

출처: OpenResponses OpenAPI, OpenResponses 사양 사이트, Hugging Face 블로그 게시물.

추출된 핵심 포인트:

- `POST /v1/responses`는 `model`, `input` (문자열 또는 `ItemParam[]`), `instructions`, `tools`,
  `tool_choice`, `stream`, `max_output_tokens`, `max_tool_calls`와 같은 `CreateResponseBody` 필드를
  수용합니다.
- `ItemParam`은 다음의 구분된 유니온입니다:
  - 역할 `system`, `developer`, `user`, `assistant`를 가진 `message` 항목
  - `function_call` 및 `function_call_output`
  - `reasoning`
  - `item_reference`
- 성공 응답은 `object: "response"`, `status`, `output` 항목이 포함된 `ResponseResource`를 반환합니다.
- 스트리밍은 다음과 같은 시맨틱 이벤트를 사용합니다:
  - `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  - `response.output_item.added`, `response.output_item.done`
  - `response.content_part.added`, `response.content_part.done`
  - `response.output_text.delta`, `response.output_text.done`
- 스펙이 요구하는 사항:
  - `Content-Type: text/event-stream`
  - `event:`는 JSON `type` 필드와 일치해야 함
  - 터미널 이벤트는 리터럴 `[DONE]`이어야 함
- Reasoning 항목은 `content`, `encrypted_content`, `summary`를 노출할 수 있습니다.
- HF 예제는 요청에 `OpenResponses-Version: latest`를 포함합니다 (선택적 헤더).

## 제안된 아키텍처

- Zod 스키마만 포함하는 `src/gateway/open-responses.schema.ts` 추가 (Gateway 임포트 없음).
- `/v1/responses`를 위한 `src/gateway/openresponses-http.ts` (또는 `open-responses-http.ts`) 추가.
- `src/gateway/openai-http.ts`를 레거시 호환성 어댑터로 그대로 유지.
- 설정 `gateway.http.endpoints.responses.enabled` 추가 (기본값 `false`).
- `gateway.http.endpoints.chatCompletions.enabled`를 독립적으로 유지; 두 엔드포인트를 별도로
  전환 가능하게 함.
- Chat Completions가 활성화되면 레거시 상태를 알리는 시작 경고를 발생.

## Chat Completions 폐기 경로

- 엄격한 모듈 경계 유지: responses와 chat completions 간에 공유 스키마 타입 없음.
- Chat Completions를 설정으로 옵트인 가능하게 하여 코드 변경 없이 비활성화 가능.
- `/v1/responses`가 안정화되면 Chat Completions를 레거시로 표시하도록 문서 업데이트.
- 선택적 향후 단계: 더 간단한 제거 경로를 위해 Chat Completions 요청을 Responses 핸들러에 매핑.

## 1단계 지원 하위 집합

- `input`을 문자열 또는 메시지 역할과 `function_call_output`이 포함된 `ItemParam[]`로 수용.
- system 및 developer 메시지를 `extraSystemPrompt`로 추출.
- 가장 최근의 `user` 또는 `function_call_output`을 에이전트 실행을 위한 현재 메시지로 사용.
- 지원되지 않는 콘텐츠 부분(이미지/파일)은 `invalid_request_error`로 거부.
- `output_text` 콘텐츠가 포함된 단일 어시스턴트 메시지 반환.
- 토큰 계정이 연결될 때까지 0 값으로 `usage` 반환.

## 유효성 검사 전략 (SDK 없음)

- 지원되는 하위 집합에 대해 Zod 스키마 구현:
  - `CreateResponseBody`
  - `ItemParam` + 메시지 콘텐츠 부분 유니온
  - `ResponseResource`
  - Gateway에서 사용하는 스트리밍 이벤트 형태
- 드리프트를 방지하고 향후 코드 생성을 허용하기 위해 단일 분리 모듈에 스키마 유지.

## 스트리밍 구현 (1단계)

- `event:` 및 `data:` 모두 포함하는 SSE 라인.
- 필수 시퀀스 (최소 실행 가능):
  - `response.created`
  - `response.output_item.added`
  - `response.content_part.added`
  - `response.output_text.delta` (필요에 따라 반복)
  - `response.output_text.done`
  - `response.content_part.done`
  - `response.completed`
  - `[DONE]`

## 테스트 및 검증 계획

- `/v1/responses`에 대한 e2e 커버리지 추가:
  - 인증 필요
  - 비스트림 응답 형태
  - 스트림 이벤트 순서 및 `[DONE]`
  - 헤더 및 `user`를 사용한 세션 라우팅
- `src/gateway/openai-http.test.ts`는 변경하지 않음.
- 수동: `stream: true`로 `/v1/responses`에 curl하여 이벤트 순서 및 터미널 `[DONE]` 확인.

## 문서 업데이트 (후속)

- `/v1/responses` 사용법 및 예제를 위한 새 문서 페이지 추가.
- `/gateway/openai-http-api`에 레거시 노트와 `/v1/responses`에 대한 포인터 업데이트.
