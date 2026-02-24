---
summary: "`openclaw devices` CLI 참조 (기기 페어링 + 토큰 순환/취소)"
read_when:
  - 기기 페어링 요청을 승인하는 경우
  - 기기 토큰을 순환하거나 취소해야 하는 경우
title: "devices"
---

# `openclaw devices`

기기 페어링 요청과 기기 범위 토큰을 관리합니다.

## 명령

### `openclaw devices list`

보류 중인 페어링 요청과 페어링된 기기를 나열합니다.

```
openclaw devices list
openclaw devices list --json
```

### `openclaw devices approve [requestId] [--latest]`

보류 중인 기기 페어링 요청을 승인합니다. `requestId`가 생략되면 OpenClaw이
가장 최근의 보류 중인 요청을 자동으로 승인합니다.

```
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### `openclaw devices reject <requestId>`

보류 중인 기기 페어링 요청을 거부합니다.

```
openclaw devices reject <requestId>
```

### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`

특정 역할의 기기 토큰을 순환합니다 (선택적으로 범위 업데이트).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `openclaw devices revoke --device <id> --role <role>`

특정 역할의 기기 토큰을 취소합니다.

```
openclaw devices revoke --device <deviceId> --role node
```

## 공통 옵션

- `--url <url>`: Gateway WebSocket URL (구성된 경우 `gateway.remote.url`이 기본값).
- `--token <token>`: Gateway 토큰 (필요한 경우).
- `--password <password>`: Gateway 비밀번호 (비밀번호 인증).
- `--timeout <ms>`: RPC 시간 제한.
- `--json`: JSON 출력 (스크립팅에 권장).

참고: `--url`을 설정하면 CLI는 설정이나 환경 자격 증명으로 대체하지 않습니다.
`--token` 또는 `--password`를 명시적으로 전달하세요. 명시적 자격 증명이 누락되면 오류가 발생합니다.

## 참고사항

- 토큰 순환은 새 토큰(민감)을 반환합니다. 비밀처럼 취급하세요.
- 이 명령에는 `operator.pairing` (또는 `operator.admin`) 범위가 필요합니다.
