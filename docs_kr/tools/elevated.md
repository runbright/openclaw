---
summary: "상승 모드 exec 및 /elevated 지시어"
read_when:
  - 상승 모드 기본값, 허용 목록 또는 슬래시 명령 동작을 조정할 때
title: "상승 모드"
---

# 상승 모드 (/elevated 지시어)

## 기능

- `/elevated on`은 게이트웨이 호스트에서 실행되며 exec 승인을 유지합니다 (`/elevated ask`와 동일).
- `/elevated full`은 게이트웨이 호스트에서 실행되며 **또한** exec를 자동 승인합니다 (exec 승인 건너뜀).
- `/elevated ask`는 게이트웨이 호스트에서 실행되지만 exec 승인을 유지합니다 (`/elevated on`과 동일).
- `on`/`ask`는 `exec.security=full`을 강제하지 **않습니다**; 설정된 보안/ask 정책이 여전히 적용됩니다.
- 에이전트가 **샌드박스**되어 있을 때만 동작을 변경합니다 (그렇지 않으면 exec가 이미 호스트에서 실행됩니다).
- 지시어 형식: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- `on|off|ask|full`만 허용됩니다; 다른 것은 힌트를 반환하고 상태를 변경하지 않습니다.

## 제어하는 것 (그리고 제어하지 않는 것)

- **가용성 게이트**: `tools.elevated`가 전역 기준입니다. `agents.list[].tools.elevated`는 에이전트별로 상승 모드를 추가로 제한할 수 있습니다 (둘 다 허용해야 함).
- **세션별 상태**: `/elevated on|off|ask|full`은 현재 세션 키에 대한 상승 수준을 설정합니다.
- **인라인 지시어**: 메시지 내 `/elevated on|ask|full`은 해당 메시지에만 적용됩니다.
- **그룹**: 그룹 채팅에서 상승 지시어는 에이전트가 멘션될 때만 적용됩니다. 멘션 요구 사항을 우회하는 명령 전용 메시지는 멘션된 것으로 처리됩니다.
- **호스트 실행**: 상승 모드는 `exec`를 게이트웨이 호스트로 강제합니다; `full`은 또한 `security=full`을 설정합니다.
- **승인**: `full`은 exec 승인을 건너뛰고; `on`/`ask`는 허용 목록/ask 규칙이 요구할 때 승인을 준수합니다.
- **비샌드박스 에이전트**: 위치에 대해 무동작; 게이팅, 로깅 및 상태에만 영향을 줍니다.
- **도구 정책은 여전히 적용됩니다**: `exec`가 도구 정책에 의해 거부된 경우, 상승 모드를 사용할 수 없습니다.
- **`/exec`와 분리**: `/exec`는 승인된 발신자에 대한 세션별 기본값을 조정하며 상승 모드를 필요로 하지 않습니다.

## 해결 순서

1. 메시지의 인라인 지시어 (해당 메시지에만 적용).
2. 세션 재정의 (지시어 전용 메시지를 전송하여 설정).
3. 전역 기본값 (설정의 `agents.defaults.elevatedDefault`).

## 세션 기본값 설정

- **지시어만** 포함된 메시지를 전송합니다 (공백 허용), 예: `/elevated full`.
- 확인 응답이 전송됩니다 (`Elevated mode set to full...` / `Elevated mode disabled.`).
- 상승 접근이 비활성화되었거나 발신자가 승인된 허용 목록에 없으면, 지시어는 실행 가능한 오류로 응답하고 세션 상태를 변경하지 않습니다.
- 인수 없이 `/elevated` (또는 `/elevated:`)를 전송하면 현재 상승 수준을 확인할 수 있습니다.

## 가용성 + 허용 목록

- 기능 게이트: `tools.elevated.enabled` (코드가 지원하더라도 설정을 통해 기본값이 off일 수 있음).
- 발신자 허용 목록: `tools.elevated.allowFrom`과 제공자별 허용 목록 (예: `discord`, `whatsapp`).
- 접두사 없는 허용 목록 항목은 발신자 범위 ID 값만 일치합니다 (`SenderId`, `SenderE164`, `From`); 수신자 라우팅 필드는 상승 권한 부여에 사용되지 않습니다.
- 변경 가능한 발신자 메타데이터에는 명시적 접두사가 필요합니다:
  - `name:<value>`는 `SenderName`과 일치
  - `username:<value>`는 `SenderUsername`과 일치
  - `tag:<value>`는 `SenderTag`와 일치
  - `id:<value>`, `from:<value>`, `e164:<value>`는 명시적 ID 타겟팅에 사용 가능
- 에이전트별 게이트: `agents.list[].tools.elevated.enabled` (선택 사항; 추가 제한만 가능).
- 에이전트별 허용 목록: `agents.list[].tools.elevated.allowFrom` (선택 사항; 설정 시 발신자가 전역 + 에이전트별 허용 목록 **모두**와 일치해야 함).
- Discord 폴백: `tools.elevated.allowFrom.discord`가 생략된 경우, `channels.discord.allowFrom` 목록이 폴백으로 사용됩니다 (레거시: `channels.discord.dm.allowFrom`). 재정의하려면 `tools.elevated.allowFrom.discord` (빈 `[]`도 가능)를 설정하세요. 에이전트별 허용 목록은 폴백을 사용하지 **않습니다**.
- 모든 게이트가 통과해야 합니다; 그렇지 않으면 상승 모드는 사용 불가로 처리됩니다.

## 로깅 + 상태

- 상승 exec 호출은 info 수준으로 로깅됩니다.
- 세션 상태에 상승 모드가 포함됩니다 (예: `elevated=ask`, `elevated=full`).
