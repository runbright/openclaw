---
summary: "채널 불가지론적 세션 바인딩 아키텍처 및 반복 1 전달 범위"
read_when:
  - 채널 불가지론적 세션 라우팅 및 바인딩 리팩토링 시
  - 채널 간 중복, 오래된 또는 누락된 세션 전달 조사 시
owner: "onutc"
status: "in-progress"
last_updated: "2026-02-21"
title: "세션 바인딩 채널 불가지론적 계획"
---

# 세션 바인딩 채널 불가지론적 계획

## 개요

이 문서는 장기적인 채널 불가지론적 세션 바인딩 모델과 다음 구현 반복의 구체적 범위를 정의합니다.

목표:

- 서브에이전트 바인딩 세션 라우팅을 핵심 기능으로 만들기
- 채널 특정 동작을 어댑터에 유지
- 일반 Discord 동작에서의 회귀 방지

## 이것이 존재하는 이유

현재 동작이 다음을 혼합합니다:

- 완료 콘텐츠 정책
- 대상 라우팅 정책
- Discord 특정 세부사항

이로 인해 다음과 같은 엣지 케이스가 발생했습니다:

- 동시 실행에서의 중복 메인 및 스레드 전달
- 재사용된 바인딩 매니저에서의 오래된 토큰 사용
- 웹훅 전송에 대한 누락된 활동 기록

## 반복 1 범위

이 반복은 의도적으로 제한됩니다.

### 1. 채널 불가지론적 핵심 인터페이스 추가

바인딩 및 라우팅을 위한 핵심 타입과 서비스 인터페이스를 추가합니다.

제안된 핵심 타입:

```ts
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

핵심 서비스 계약:

```ts
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2. 서브에이전트 완료를 위한 하나의 핵심 전달 라우터 추가

완료 이벤트에 대한 단일 대상 해석 경로를 추가합니다.

라우터 계약:

```ts
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

이 반복에서:

- `task_completion`만 이 새 경로를 통해 라우팅됨
- 다른 이벤트 종류에 대한 기존 경로는 그대로 유지

### 3. Discord를 어댑터로 유지

Discord는 첫 번째 어댑터 구현으로 유지됩니다.

어댑터 책임:

- 스레드 대화 생성/재사용
- 웹훅 또는 채널 전송을 통한 바인딩 메시지 전송
- 스레드 상태 유효성 검사 (보관됨/삭제됨)
- 어댑터 메타데이터 매핑 (웹훅 ID, 스레드 ID)

### 4. 현재 알려진 정확성 문제 수정

이 반복에서 필수:

- 기존 스레드 바인딩 매니저 재사용 시 토큰 사용 갱신
- 웹훅 기반 Discord 전송에 대한 아웃바운드 활동 기록
- 세션 모드 완료를 위해 바인딩된 스레드 대상이 선택된 경우 암묵적 메인 채널 폴백 중지

### 5. 현재 런타임 안전 기본값 보존

스레드 바인딩 spawn이 비활성화된 사용자에 대해 동작 변경 없음.

기본값 유지:

- `channels.discord.threadBindings.spawnSubagentSessions = false`

결과:

- 일반 Discord 사용자는 현재 동작 유지
- 새 핵심 경로는 활성화된 곳에서 바인딩된 세션 완료 라우팅에만 영향

## 반복 1에 포함되지 않는 항목

명시적으로 연기됨:

- ACP 바인딩 대상 (`targetKind: "acp"`)
- Discord 이외의 새 채널 어댑터
- 모든 전달 경로의 글로벌 대체 (`spawn_ack`, 향후 `subagent_message`)
- 프로토콜 수준 변경
- 모든 바인딩 지속성을 위한 저장소 마이그레이션/버전 관리 재설계

ACP에 대한 참고:

- 인터페이스 설계는 ACP를 위한 여지를 남김
- ACP 구현은 이 반복에서 시작되지 않음

## 라우팅 불변식

이 불변식들은 반복 1에서 필수입니다.

- 대상 선택과 콘텐츠 생성은 별도의 단계
- 세션 모드 완료가 활성 바인딩 대상으로 해석되면 전달은 해당 대상을 타겟으로 해야 함
- 바인딩 대상에서 메인 채널로의 숨겨진 재라우팅 없음
- 폴백 동작은 명시적이고 관측 가능해야 함

## 호환성 및 롤아웃

호환성 목표:

- 스레드 바인딩 spawn이 꺼진 사용자에 대해 회귀 없음
- 이 반복에서 비-Discord 채널에 대한 변경 없음

롤아웃:

1. 현재 기능 게이트 뒤에 인터페이스와 라우터 배치.
2. Discord 완료 모드 바인딩 전달을 라우터를 통해 라우팅.
3. 비바인딩 흐름에 대한 레거시 경로 유지.
4. 대상 테스트 및 카나리아 런타임 로그로 검증.

## 반복 1에서 필요한 테스트

단위 및 통합 커버리지 필수:

- 매니저 재사용 후 매니저 토큰 순환이 최신 토큰 사용
- 웹훅 전송이 채널 활동 타임스탬프 업데이트
- 동일한 요청자 채널의 두 활성 바인딩 세션이 메인 채널에 중복되지 않음
- 바인딩 세션 모드 실행에 대한 완료가 스레드 대상으로만 해석됨
- 비활성화된 spawn 플래그가 레거시 동작을 변경 없이 유지

## 제안된 구현 파일

핵심:

- `src/infra/outbound/session-binding-service.ts` (신규)
- `src/infra/outbound/bound-delivery-router.ts` (신규)
- `src/agents/subagent-announce.ts` (완료 대상 해석 통합)

Discord 어댑터 및 런타임:

- `src/discord/monitor/thread-bindings.manager.ts`
- `src/discord/monitor/reply-delivery.ts`
- `src/discord/send.outbound.ts`

테스트:

- `src/discord/monitor/provider*.test.ts`
- `src/discord/monitor/reply-delivery.test.ts`
- `src/agents/subagent-announce.format.test.ts`

## 반복 1 완료 기준

- 핵심 인터페이스가 존재하고 완료 라우팅에 연결됨
- 위의 정확성 수정이 테스트와 함께 병합됨
- 세션 모드 바인딩 실행에서 메인 및 스레드 중복 완료 전달이 없음
- 비활성화된 바인딩 spawn 배포에서 동작 변경 없음
- ACP는 명시적으로 연기됨
