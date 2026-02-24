---
summary: "`openclaw system` (시스템 이벤트, 하트비트, 프레즌스) CLI 레퍼런스"
read_when:
  - cron 작업을 만들지 않고 시스템 이벤트를 대기열에 넣고 싶을 때
  - 하트비트를 활성화하거나 비활성화해야 할 때
  - 시스템 프레즌스 항목을 검사하고 싶을 때
title: "system"
---

# `openclaw system`

Gateway를 위한 시스템 레벨 헬퍼: 시스템 이벤트 대기열, 하트비트 제어,
프레즌스 확인.

## 일반 명령어

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

**기본** 세션에 시스템 이벤트를 대기열에 넣습니다. 다음 하트비트가
프롬프트에 `System:` 라인으로 주입합니다. `--mode now`를 사용하면 즉시 하트비트를
트리거합니다; `next-heartbeat`은 다음 예약된 틱을 기다립니다.

플래그:

- `--text <text>`: 필수 시스템 이벤트 텍스트.
- `--mode <mode>`: `now` 또는 `next-heartbeat` (기본값).
- `--json`: 머신 판독 가능 출력.

## `system heartbeat last|enable|disable`

하트비트 제어:

- `last`: 마지막 하트비트 이벤트를 표시합니다.
- `enable`: 하트비트를 다시 켭니다 (비활성화된 경우 사용).
- `disable`: 하트비트를 일시 중지합니다.

플래그:

- `--json`: 머신 판독 가능 출력.

## `system presence`

Gateway가 알고 있는 현재 시스템 프레즌스 항목을 나열합니다 (노드,
인스턴스 및 유사한 상태 라인).

플래그:

- `--json`: 머신 판독 가능 출력.

## 참고

- 현재 구성(로컬 또는 원격)으로 접근 가능한 실행 중인 Gateway가 필요합니다.
- 시스템 이벤트는 임시적이며 재시작 시 유지되지 않습니다.
