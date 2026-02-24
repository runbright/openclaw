---
summary: "계획: CDP를 사용하여 브라우저 act:evaluate를 Playwright 큐에서 분리, 종단 간 데드라인 및 안전한 ref 해석"
read_when:
  - 브라우저 `act:evaluate` 타임아웃, 중단 또는 큐 차단 문제 작업 시
  - evaluate 실행을 위한 CDP 기반 분리 계획 시
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "브라우저 Evaluate CDP 리팩토링"
---

# 브라우저 Evaluate CDP 리팩토링 계획

## 컨텍스트

`act:evaluate`는 사용자가 제공한 JavaScript를 페이지에서 실행합니다. 현재는 Playwright
(`page.evaluate` 또는 `locator.evaluate`)를 통해 실행됩니다. Playwright는 페이지별로 CDP 명령을
직렬화하므로, 멈추거나 오래 실행되는 evaluate는 페이지 명령 큐를 차단하고 해당 탭의 이후 모든 작업이
"멈춘" 것처럼 보이게 할 수 있습니다.

PR #13498은 실용적인 안전망(제한된 evaluate, 중단 전파, 최선 노력 복구)을 추가합니다. 이 문서는
`act:evaluate`를 Playwright에서 본질적으로 분리하여 멈춘 evaluate가 정상적인 Playwright 작업을
방해할 수 없게 하는 더 큰 리팩토링을 설명합니다.

## 목표

- `act:evaluate`는 동일한 탭의 이후 브라우저 작업을 영구적으로 차단할 수 없어야 합니다.
- 타임아웃은 종단 간 단일 소스로 작동하여 호출자가 예산에 의존할 수 있어야 합니다.
- 중단과 타임아웃은 HTTP 및 인프로세스 디스패치 전반에서 동일한 방식으로 처리됩니다.
- evaluate에 대한 요소 타겟팅이 모든 것을 Playwright에서 전환하지 않고도 지원됩니다.
- 기존 호출자 및 페이로드에 대한 하위 호환성을 유지합니다.

## 비목표

- 모든 브라우저 작업(click, type, wait 등)을 CDP 구현으로 대체.
- PR #13498에서 도입된 기존 안전망 제거(유용한 폴백으로 유지됨).
- 기존 `browser.evaluateEnabled` 게이트를 넘어서는 새로운 안전하지 않은 기능 도입.
- evaluate에 대한 프로세스 격리(워커 프로세스/스레드) 추가. 이 리팩토링 후에도 복구하기 어려운
  멈춤 상태가 계속 발생하면 후속 아이디어로 검토합니다.

## 현재 아키텍처 (왜 멈추는가)

개략적으로:

- 호출자가 `act:evaluate`를 브라우저 제어 서비스에 보냅니다.
- 라우트 핸들러가 Playwright를 호출하여 JavaScript를 실행합니다.
- Playwright가 페이지 명령을 직렬화하므로 완료되지 않는 evaluate가 큐를 차단합니다.
- 차단된 큐는 해당 탭의 이후 click/type/wait 작업이 중단된 것처럼 보이게 합니다.

## 제안된 아키텍처

### 1. 데드라인 전파

단일 예산 개념을 도입하고 모든 것을 여기에서 파생합니다:

- 호출자가 `timeoutMs`(또는 미래의 데드라인)를 설정합니다.
- 외부 요청 타임아웃, 라우트 핸들러 로직, 페이지 내 실행 예산 모두 동일한 예산을 사용하며,
  직렬화 오버헤드를 위한 작은 여유가 필요한 곳에 추가됩니다.
- 중단은 모든 곳에서 `AbortSignal`로 전파되어 취소가 일관적입니다.

구현 방향:

- 작은 헬퍼(예: `createBudget({ timeoutMs, signal })`)를 추가하여 반환:
  - `signal`: 연결된 AbortSignal
  - `deadlineAtMs`: 절대 데드라인
  - `remainingMs()`: 하위 작업을 위한 남은 예산
- 이 헬퍼를 다음에서 사용:
  - `src/browser/client-fetch.ts` (HTTP 및 인프로세스 디스패치)
  - `src/node-host/runner.ts` (프록시 경로)
  - 브라우저 액션 구현 (Playwright 및 CDP)

### 2. 별도의 Evaluate 엔진 (CDP 경로)

Playwright의 페이지별 명령 큐를 공유하지 않는 CDP 기반 evaluate 구현을 추가합니다. 핵심 속성은
evaluate 전송이 별도의 WebSocket 연결이며 대상에 연결된 별도의 CDP 세션이라는 것입니다.

구현 방향:

- 새 모듈, 예: `src/browser/cdp-evaluate.ts`:
  - 구성된 CDP 엔드포인트(브라우저 수준 소켓)에 연결합니다.
  - `Target.attachToTarget({ targetId, flatten: true })`를 사용하여 `sessionId`를 얻습니다.
  - 다음 중 하나를 실행합니다:
    - 페이지 수준 evaluate의 경우 `Runtime.evaluate`, 또는
    - 요소 evaluate의 경우 `DOM.resolveNode` 플러스 `Runtime.callFunctionOn`.
  - 타임아웃 또는 중단 시:
    - 세션에 대해 `Runtime.terminateExecution`을 최선 노력으로 전송합니다.
    - WebSocket을 닫고 명확한 오류를 반환합니다.

참고:

- 이것은 여전히 페이지에서 JavaScript를 실행하므로 종료에 부작용이 있을 수 있습니다. 이점은
  Playwright 큐를 방해하지 않으며, CDP 세션을 종료하여 전송 레이어에서 취소 가능하다는 것입니다.

### 3. Ref 처리 (전체 재작성 없이 요소 타겟팅)

어려운 부분은 요소 타겟팅입니다. CDP는 DOM 핸들 또는 `backendDOMNodeId`가 필요하지만,
현재 대부분의 브라우저 작업은 스냅샷의 ref를 기반으로 한 Playwright 로케이터를 사용합니다.

권장 접근 방식: 기존 ref를 유지하되 선택적 CDP 해석 가능 id를 첨부합니다.

#### 3.1 저장된 Ref 정보 확장

저장된 역할 ref 메타데이터를 선택적으로 CDP id를 포함하도록 확장합니다:

- 현재: `{ role, name, nth }`
- 제안: `{ role, name, nth, backendDOMNodeId?: number }`

이렇게 하면 기존의 모든 Playwright 기반 작업이 계속 작동하고, `backendDOMNodeId`가 사용 가능한 경우
CDP evaluate가 동일한 `ref` 값을 수용할 수 있습니다.

#### 3.2 스냅샷 시점에 backendDOMNodeId 채우기

역할 스냅샷을 생성할 때:

1. 기존과 같이 역할 ref 맵(role, name, nth)을 생성합니다.
2. CDP(`Accessibility.getFullAXTree`)를 통해 AX 트리를 가져오고 동일한 중복 처리 규칙을 사용하여
   `(role, name, nth) -> backendDOMNodeId`의 병렬 맵을 계산합니다.
3. 현재 탭의 저장된 ref 정보에 id를 다시 병합합니다.

ref에 대한 매핑이 실패하면 `backendDOMNodeId`를 undefined로 둡니다. 이렇게 하면 기능이
최선 노력이며 안전하게 배포할 수 있습니다.

#### 3.3 Ref를 사용한 Evaluate 동작

`act:evaluate`에서:

- `ref`가 있고 `backendDOMNodeId`가 있으면 CDP를 통해 요소 evaluate를 실행합니다.
- `ref`가 있지만 `backendDOMNodeId`가 없으면 Playwright 경로(안전망 포함)로 폴백합니다.

선택적 탈출구:

- 고급 호출자(및 디버깅)를 위해 `backendDOMNodeId`를 직접 수용하도록 요청 형태를 확장하되,
  `ref`를 기본 인터페이스로 유지합니다.

### 4. 최후의 복구 경로 유지

CDP evaluate를 사용하더라도 탭이나 연결을 방해하는 다른 방법이 있습니다. 다음을 위한 최후 수단으로
기존 복구 메커니즘(실행 종료 + Playwright 연결 해제)을 유지합니다:

- 레거시 호출자
- CDP 연결이 차단된 환경
- 예상치 못한 Playwright 엣지 케이스

## 구현 계획 (단일 반복)

### 산출물

- Playwright 페이지별 명령 큐 외부에서 실행되는 CDP 기반 evaluate 엔진.
- 호출자와 핸들러가 일관되게 사용하는 단일 종단 간 타임아웃/중단 예산.
- 요소 evaluate를 위해 선택적으로 `backendDOMNodeId`를 가질 수 있는 Ref 메타데이터.
- `act:evaluate`는 가능한 경우 CDP 엔진을 선호하고, 불가능한 경우 Playwright로 폴백.
- 멈춘 evaluate가 이후 작업을 방해하지 않음을 증명하는 테스트.
- 실패 및 폴백을 가시화하는 로그/메트릭.

### 구현 체크리스트

1. `timeoutMs` + 업스트림 `AbortSignal`을 연결하는 공유 "예산" 헬퍼 추가:
   - 단일 `AbortSignal`
   - 절대 데드라인
   - 다운스트림 작업을 위한 `remainingMs()` 헬퍼
2. 모든 호출자 경로에서 `timeoutMs`가 모든 곳에서 같은 의미를 갖도록 해당 헬퍼 사용:
   - `src/browser/client-fetch.ts` (HTTP 및 인프로세스 디스패치)
   - `src/node-host/runner.ts` (노드 프록시 경로)
   - `/act`를 호출하는 CLI 래퍼 (`browser evaluate`에 `--timeout-ms` 추가)
3. `src/browser/cdp-evaluate.ts` 구현:
   - 브라우저 수준 CDP 소켓에 연결
   - `Target.attachToTarget`로 `sessionId` 획득
   - 페이지 evaluate를 위해 `Runtime.evaluate` 실행
   - 요소 evaluate를 위해 `DOM.resolveNode` + `Runtime.callFunctionOn` 실행
   - 타임아웃/중단 시: 최선 노력 `Runtime.terminateExecution` 후 소켓 닫기
4. 저장된 역할 ref 메타데이터를 선택적으로 `backendDOMNodeId`를 포함하도록 확장:
   - Playwright 작업을 위한 기존 `{ role, name, nth }` 동작 유지
   - CDP 요소 타겟팅을 위한 `backendDOMNodeId?: number` 추가
5. 스냅샷 생성 중 `backendDOMNodeId` 채우기 (최선 노력):
   - CDP(`Accessibility.getFullAXTree`)를 통해 AX 트리 가져오기
   - `(role, name, nth) -> backendDOMNodeId` 계산 및 저장된 ref 맵에 병합
   - 매핑이 모호하거나 누락된 경우 id를 undefined로 두기
6. `act:evaluate` 라우팅 업데이트:
   - `ref`가 없는 경우: 항상 CDP evaluate 사용
   - `ref`가 `backendDOMNodeId`로 해석되는 경우: CDP 요소 evaluate 사용
   - 그 외: Playwright evaluate로 폴백 (여전히 제한 및 중단 가능)
7. 기존 "최후 수단" 복구 경로를 폴백으로 유지, 기본 경로가 아님.
8. 테스트 추가:
   - 멈춘 evaluate가 예산 내에서 타임아웃되고 다음 click/type가 성공함
   - 중단이 evaluate를 취소(클라이언트 연결 해제 또는 타임아웃)하고 후속 작업을 차단 해제함
   - 매핑 실패가 깔끔하게 Playwright로 폴백됨
9. 관측 가능성 추가:
   - evaluate 기간 및 타임아웃 카운터
   - terminateExecution 사용량
   - 폴백 비율 (CDP -> Playwright) 및 이유

### 수락 기준

- 의도적으로 중단된 `act:evaluate`가 호출자 예산 내에서 반환되고 이후 작업을 위해 탭을
  방해하지 않음.
- `timeoutMs`가 CLI, 에이전트 도구, 노드 프록시, 인프로세스 호출 전반에서 일관되게 동작함.
- `ref`를 `backendDOMNodeId`로 매핑할 수 있으면 요소 evaluate가 CDP를 사용; 그렇지 않으면
  폴백 경로가 여전히 제한 및 복구 가능함.

## 테스트 계획

- 단위 테스트:
  - 역할 ref와 AX 트리 노드 간 `(role, name, nth)` 매칭 로직.
  - 예산 헬퍼 동작 (여유, 남은 시간 계산).
- 통합 테스트:
  - CDP evaluate 타임아웃이 예산 내에서 반환되고 다음 작업을 차단하지 않음.
  - 중단이 evaluate를 취소하고 최선 노력으로 종료를 트리거함.
- 계약 테스트:
  - `BrowserActRequest`와 `BrowserActResponse`가 호환됨을 확인.

## 위험 및 완화

- 매핑이 불완전함:
  - 완화: 최선 노력 매핑, Playwright evaluate로 폴백, 디버그 도구 추가.
- `Runtime.terminateExecution`에 부작용이 있음:
  - 완화: 타임아웃/중단 시에만 사용하고 오류에 동작을 문서화.
- 추가 오버헤드:
  - 완화: 스냅샷 요청 시에만 AX 트리를 가져오고, 대상별로 캐시하며, CDP 세션을 짧게 유지.
- 확장 릴레이 제한:
  - 완화: 페이지별 소켓을 사용할 수 없을 때 브라우저 수준 연결 API를 사용하고,
    현재 Playwright 경로를 폴백으로 유지.

## 미해결 질문

- 새 엔진을 `playwright`, `cdp` 또는 `auto`로 구성 가능하게 해야 하는가?
- 고급 사용자를 위해 새로운 "nodeRef" 형식을 노출할 것인가, 아니면 `ref`만 유지할 것인가?
- 프레임 스냅샷과 셀렉터 범위 스냅샷이 AX 매핑에 어떻게 참여해야 하는가?
