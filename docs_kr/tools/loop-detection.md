---
title: "도구 루프 감지"
description: "반복적이거나 정체된 도구 호출 루프를 방지하기 위한 선택적 가드레일 구성"
summary: "반복적인 도구 호출 루프를 감지하는 가드레일을 활성화하고 조정하는 방법"
read_when:
  - 에이전트가 도구 호출을 반복하며 멈추는 문제가 보고된 경우
  - 반복 호출 보호 기능을 조정해야 하는 경우
  - 에이전트 도구/런타임 정책을 편집하는 경우
---

# 도구 루프 감지

OpenClaw는 에이전트가 반복적인 도구 호출 패턴에 갇히는 것을 방지할 수 있습니다.
이 가드는 **기본적으로 비활성화**되어 있습니다.

엄격한 설정에서는 정당한 반복 호출을 차단할 수 있으므로 필요한 경우에만 활성화하세요.

## 이 기능이 존재하는 이유

- 진행되지 않는 반복적인 시퀀스를 감지합니다.
- 고빈도 무결과 루프(동일한 도구, 동일한 입력, 반복된 오류)를 감지합니다.
- 알려진 폴링 도구에 대한 특정 반복 호출 패턴을 감지합니다.

## 구성 블록

전역 기본값:

```json5
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 20,
      detectorCooldownMs: 12000,
      repeatThreshold: 3,
      criticalThreshold: 6,
      detectors: {
        repeatedFailure: true,
        knownPollLoop: true,
        repeatingNoProgress: true,
      },
    },
  },
}
```

에이전트별 재정의(선택 사항):

```json5
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            repeatThreshold: 2,
            criticalThreshold: 5,
          },
        },
      },
    ],
  },
}
```

### 필드 동작

- `enabled`: 마스터 스위치. `false`이면 루프 감지가 수행되지 않습니다.
- `historySize`: 분석을 위해 보관하는 최근 도구 호출 수.
- `detectorCooldownMs`: 진행 없음 감지기에서 사용하는 시간 창.
- `repeatThreshold`: 경고/차단이 시작되기 전 최소 반복 횟수.
- `criticalThreshold`: 더 엄격한 처리를 트리거할 수 있는 강화된 임계값.
- `detectors.repeatedFailure`: 동일한 호출 경로에서 반복 실패 시도를 감지합니다.
- `detectors.knownPollLoop`: 알려진 폴링 유사 루프를 감지합니다.
- `detectors.repeatingNoProgress`: 상태 변경 없이 고빈도 반복 호출을 감지합니다.

## 권장 설정

- `enabled: true`로 시작하고 기본값을 변경하지 않습니다.
- 오탐이 발생하는 경우:
  - `repeatThreshold` 및/또는 `criticalThreshold`를 높이세요
  - 문제를 일으키는 감지기만 비활성화하세요
  - 덜 엄격한 이력 컨텍스트를 위해 `historySize`를 줄이세요

## 로그 및 예상 동작

루프가 감지되면 OpenClaw는 루프 이벤트를 보고하고 심각도에 따라 다음 도구 사이클을 차단하거나 억제합니다.
이는 정상적인 도구 접근을 유지하면서 사용자를 폭주하는 토큰 소비와 잠금으로부터 보호합니다.

- 먼저 경고와 임시 억제를 선호합니다.
- 반복된 증거가 축적될 때만 에스컬레이션합니다.

## 참고 사항

- `tools.loopDetection`은 에이전트 수준 재정의와 병합됩니다.
- 에이전트별 구성은 전역 값을 완전히 재정의하거나 확장합니다.
- 구성이 없으면 가드레일은 비활성 상태를 유지합니다.
