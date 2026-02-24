---
summary: "게이트웨이 프로토콜의 단일 정보 소스로서의 TypeBox 스키마 활용 가이드"
read_when:
  - 프로토콜 스케마나 코드 생성을 업데이트할 때
title: "TypeBox"
---

# 프로토콜의 단일 정보 소스: TypeBox

OpenClaw는 **게이트웨이 WebSocket 프로토콜**(핸드셰이크, 요청/응답, 서버 이벤트)을 정의하기 위해 TypeScript 기반의 스키마 라이브러리인 TypeBox를 사용합니다.

이 스키마는 **런타임 검증**, **JSON 스키마 내보내기**, 그리고 macOS 앱을 위한 **Swift 코드 생성**의 기초가 됩니다. 즉, 하나의 소스(TypeBox 스키마)만 수정하면 다른 모든 부분은 자동으로 생성됩니다.

## 통신 모델 (WebSocket 프레임)

모든 게이트웨이 WebSocket 메시지는 다음 세 가지 프레임 중 하나로 구성됩니다:

- **요청 (Request)**: `{ type: "req", id, method, params }` - 클라이언트에서 서버로
- **응답 (Response)**: `{ type: "res", id, ok, payload | error }` - 서버에서 클라이언트로
- **이벤트 (Event)**: `{ type: "event", event, payload, ... }` - 서버에서 클라이언트로 (푸시 알림)

연결 시 가장 먼저 `connect` 요청을 보내야 하며, 이후 `health`, `send`, `chat.send` 등의 메서드를 호출하거나 `presence`, `agent` 등의 이벤트를 구독할 수 있습니다.

## 주요 관리 경로

- **스키마 소스**: `src/gateway/protocol/schema.ts`
- **런타임 검증기**: `src/gateway/protocol/index.ts`
- **서버 구현**: `src/gateway/server.ts`
- **생성된 JSON 스키마**: `dist/protocol.schema.json`
- **생성된 Swift 모델**: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## 작동 원리

1. **서버 측**: 클라이언트로부터 들어오는 모든 데이터 프레임은 AJV(Another JSON Schema Validator)를 통해 스키마에 맞는지 검증됩니다. 형식이 맞지 않는 데이터는 즉시 거부됩니다.
2. **클라이언트 측**: JS 클라이언트나 Swift 클라이언트는 서버에서 오는 이벤트와 응답을 사용하기 전에 검증합니다.
3. **코드 생성**: `pnpm protocol:check` 명령을 실행하면 JSON 스키마와 Swift 모델 코드가 현재 TypeBox 정의에 맞게 최신화됩니다.

## 새로운 프로토콜 메서드 추가 과정

1. `src/gateway/protocol/schema.ts`에 새로운 요청/응답 스키마를 정의합니다.
2. `src/gateway/protocol/index.ts`에 해당 스키마에 대한 유효성 검사기(Validator)를 추가합니다.
3. `src/gateway/server-methods/` 하위에 실제 서버 동작(Handler)을 구현합니다.
4. `pnpm protocol:check`를 실행하여 관련 파일들을 재생성하고 커밋합니다.

## 프로토콜 설계 원칙

- **엄격한 페이로드**: 대부분의 객체는 `additionalProperties: false`를 사용하여 정해진 필드 외의 데이터 유입을 차단합니다.
- **멱등성 보장**: 부수 효과(메시지 전송, 투표 등)가 있는 메서드는 중복 요청을 방지하기 위해 `idempotencyKey`를 요구합니다.
- **버전 관리**: 클라이언트는 연결 시 사용할 프로토콜 버전을 명시하며, 서버는 지원하지 않는 버전의 연결을 거부합니다.

상세한 프레임 예시나 단계별 튜토리얼은 [영문 원본 문서](/concepts/typebox)를 참고하십시오.
---
