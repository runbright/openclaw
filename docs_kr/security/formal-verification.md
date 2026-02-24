---
title: 형식 검증 (보안 모델)
summary: OpenClaw의 최고 위험 경로에 대한 기계 검증 보안 모델.
read_when:
  - Reviewing formal security model guarantees or limits
  - Reproducing or updating TLA+/TLC security model checks
permalink: /security/formal-verification/
---

# 형식 검증 (보안 모델)

이 페이지는 OpenClaw의 **형식 보안 모델** (현재 TLA+/TLC; 필요에 따라 추가)을 추적합니다.

> 참고: 일부 이전 링크는 이전 프로젝트 이름을 참조할 수 있습니다.

**목표 (궁극적 방향):** 명시적 가정 하에서 OpenClaw가 의도된 보안 정책(인가, 세션 격리, 도구 게이팅 및 잘못된 구성 안전성)을 시행한다는 기계 검증된 논거를 제공합니다.

**현재 상태:** 실행 가능한 공격자 주도의 **보안 회귀 테스트 스위트**입니다:

- 각 주장에는 유한 상태 공간에 대한 실행 가능한 모델 검사가 있습니다.
- 많은 주장에는 현실적인 버그 클래스에 대한 반례 추적을 생성하는 쌍을 이루는 **부정 모델**이 있습니다.

**아직 아닌 것:** "OpenClaw가 모든 측면에서 안전하다"는 증명이나 전체 TypeScript 구현이 올바르다는 증명이 아닙니다.

## 모델 위치

모델은 별도의 저장소에서 관리됩니다: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

## 중요한 주의사항

- 이것은 전체 TypeScript 구현이 아닌 **모델**입니다. 모델과 코드 간의 불일치가 가능합니다.
- 결과는 TLC가 탐색한 상태 공간에 의해 제한됩니다; "그린"이라고 해서 모델링된 가정과 범위를 넘어선 보안을 의미하지 않습니다.
- 일부 주장은 명시적 환경 가정(예: 올바른 배포, 올바른 구성 입력)에 의존합니다.

## 결과 재현

현재, 결과는 모델 저장소를 로컬에 클론하고 TLC를 실행하여 재현합니다(아래 참조). 향후 반복에서는 다음을 제공할 수 있습니다:

- 공개 아티팩트(반례 추적, 실행 로그)가 포함된 CI 실행 모델
- 작은 범위의 검사를 위한 호스팅된 "이 모델 실행" 워크플로우

시작하기:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ 필요 (TLC는 JVM에서 실행됩니다).
# 저장소에는 고정된 `tla2tools.jar` (TLA+ 도구)가 포함되어 있으며 `bin/tlc` + Make 타겟을 제공합니다.

make <target>
```

### Gateway 노출 및 open gateway 잘못된 구성

**주장:** loopback을 넘어서 인증 없이 바인딩하면 원격 침해가 가능해질 수 있으며 / 노출이 증가합니다; token/password는 비인증 공격자를 차단합니다(모델 가정에 따라).

- 그린 실행:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 레드 (예상됨):
  - `make gateway-exposure-v2-negative`

참고: 모델 저장소의 `docs/gateway-exposure-matrix.md`.

### Nodes.run 파이프라인 (최고 위험 기능)

**주장:** `nodes.run`은 (a) 노드 명령 허용 목록과 선언된 명령 및 (b) 구성된 경우 실시간 승인이 필요합니다; 승인은 재생을 방지하기 위해 토큰화됩니다(모델 내에서).

- 그린 실행:
  - `make nodes-pipeline`
  - `make approvals-token`
- 레드 (예상됨):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### 페어링 저장소 (DM 게이팅)

**주장:** 페어링 요청은 TTL과 보류 요청 상한을 준수합니다.

- 그린 실행:
  - `make pairing`
  - `make pairing-cap`
- 레드 (예상됨):
  - `make pairing-negative`
  - `make pairing-cap-negative`

### 수신 게이팅 (멘션 + 제어 명령 우회)

**주장:** 멘션이 필요한 그룹 컨텍스트에서, 권한 없는 "제어 명령"은 멘션 게이팅을 우회할 수 없습니다.

- 그린:
  - `make ingress-gating`
- 레드 (예상됨):
  - `make ingress-gating-negative`

### 라우팅/세션 키 격리

**주장:** 서로 다른 피어의 DM은 명시적으로 연결/구성되지 않는 한 동일한 세션으로 합쳐지지 않습니다.

- 그린:
  - `make routing-isolation`
- 레드 (예상됨):
  - `make routing-isolation-negative`

## v1++: 추가 범위 모델 (동시성, 재시도, 추적 정확성)

실제 실패 모드(비원자적 업데이트, 재시도, 메시지 팬아웃) 주변의 충실도를 높이는 후속 모델입니다.

### 페어링 저장소 동시성 / 멱등성

**주장:** 페어링 저장소는 인터리빙 하에서도 `MaxPending`과 멱등성을 시행해야 합니다(즉, "검사-후-쓰기"는 원자적/잠금이어야 합니다; 새로고침이 중복을 생성해서는 안 됩니다).

이것이 의미하는 것:

- 동시 요청 하에서, 채널에 대한 `MaxPending`을 초과할 수 없습니다.
- 동일한 `(channel, sender)`에 대한 반복 요청/새로고침이 중복된 활성 보류 행을 생성해서는 안 됩니다.

- 그린 실행:
  - `make pairing-race` (원자적/잠금 상한 검사)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 레드 (예상됨):
  - `make pairing-race-negative` (비원자적 begin/commit 상한 경쟁)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### 수신 추적 상관관계 / 멱등성

**주장:** 수집은 팬아웃 전반에 걸쳐 추적 상관관계를 보존해야 하며, 제공자 재시도 하에서 멱등적이어야 합니다.

이것이 의미하는 것:

- 하나의 외부 이벤트가 여러 내부 메시지가 될 때, 모든 부분이 동일한 추적/이벤트 식별자를 유지합니다.
- 재시도가 이중 처리를 발생시키지 않습니다.
- 제공자 이벤트 ID가 없는 경우, 중복 제거는 안전한 키(예: 추적 ID)로 폴백하여 고유 이벤트를 삭제하지 않습니다.

- 그린:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- 레드 (예상됨):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### 라우팅 dmScope 우선순위 + identityLinks

**주장:** 라우팅은 기본적으로 DM 세션을 격리된 상태로 유지해야 하며, 명시적으로 구성된 경우에만 세션을 합칩니다(채널 우선순위 + identity links).

이것이 의미하는 것:

- 채널별 dmScope 오버라이드가 전역 기본값보다 우선해야 합니다.
- identityLinks는 명시적으로 연결된 그룹 내에서만 합쳐야 하며, 관련 없는 피어 간에는 합쳐서는 안 됩니다.

- 그린:
  - `make routing-precedence`
  - `make routing-identitylinks`
- 레드 (예상됨):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`
