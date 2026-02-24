---
summary: "에이전트, 메시지 헤더 및 시스템 프롬프트를 위한 타임존(시간대) 처리 가이드"
read_when:
  - 모델에 전달되는 타임스탬프가 어떻게 정규화되는지 이해해야 할 때
  - 시스템 프롬프트를 위한 사용자 타임존을 설정할 때
title: "타임존"
---

# 타임존 (Timezones)

OpenClaw는 모델이 **단일 기준 시간**을 볼 수 있도록 타임스탬프를 표준화합니다.

## 메시지 헤더 (기본값: 서버 현지 시간)

입력되는 메시지는 다음과 같은 헤더 정보와 함께 모델에 전달됩니다:

```
[Telegram 홍길동 ... 2026-01-05 16:26 KST] 안녕하세요
```

메시지 헤더의 타임스탬프는 기본적으로 **게이트웨이 서버의 현지 시간**을 따릅니다. 설정을 통해 이를 변경할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA 타임존 (예: "Asia/Seoul")
      envelopeTimestamp: "on", // 절대 시간 표시 여부
      envelopeElapsed: "on", // 이전 메시지로부터 경과된 시간 표시 여부
    },
  },
}
```

- `utc`: 협정 세계시(UTC)를 사용합니다.
- `user`: `agents.defaults.userTimezone` 설정을 따릅니다 (설정이 없으면 서버 시간 사용).
- `Asia/Seoul`: 특정 IANA 타임존을 강제로 지정합니다.

## 시스템 프롬프트용 사용자 타임존

에이전트에게 사용자의 현지 시간을 알리려면 `agents.defaults.userTimezone`을 설정하십시오. 이 설정이 없으면 OpenClaw는 실행 시점의 **서버 타임존**을 기반으로 시간을 계산합니다.

```json5
{
  agents: { defaults: { userTimezone: "Asia/Seoul" } },
}
```

시스템 프롬프트에는 다음과 같은 정보가 포함되어 모델이 현재 시간을 인지하게 됩니다:
- `Current Date & Time`: 현지 날짜 및 시간과 타임존 정보
- `Time format`: 12시간제 또는 24시간제 표시 방식

## 도구 실행 결과 (Tool payloads)

도구 호출을 통해 가져오는 메시지 목록 등에는 제공자(Telegram, Slack 등)의 원본 타임스탬프와 함께 일관성을 위한 정규화된 필드가 추가됩니다:
- `timestampMs`: UTC 기준 밀리초(ms)
- `timestampUtc`: ISO 8601 UTC 문자열

상세한 동작 방식은 [영문 원본 문서](/concepts/timezone)를 참고하십시오.
---
