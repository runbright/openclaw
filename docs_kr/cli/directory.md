---
summary: "`openclaw directory` CLI 참조 (자신, 연락처, 그룹)"
read_when:
  - 채널의 연락처/그룹/자신의 ID를 조회하려는 경우
  - 채널 디렉토리 어댑터를 개발하는 경우
title: "directory"
---

# `openclaw directory`

지원되는 채널에 대한 디렉토리 조회 (연락처/피어, 그룹, "나").

## 공통 플래그

- `--channel <name>`: 채널 ID/별칭 (여러 채널이 구성된 경우 필수; 하나만 구성된 경우 자동)
- `--account <id>`: 계정 ID (기본값: 채널 기본값)
- `--json`: JSON 출력

## 참고사항

- `directory`는 다른 명령(특히 `openclaw message send --target ...`)에 붙여넣을 수 있는 ID를 찾는 데 도움을 주기 위한 것입니다.
- 많은 채널에서 결과는 실시간 프로바이더 디렉토리가 아닌 설정 기반(허용 목록 / 구성된 그룹)입니다.
- 기본 출력은 탭으로 구분된 `id`(및 경우에 따라 `name`)입니다. 스크립팅에는 `--json`을 사용합니다.

## `message send`와 함께 결과 사용

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## ID 형식 (채널별)

- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (그룹)
- Telegram: `@username` 또는 숫자 채팅 ID; 그룹은 숫자 ID
- Slack: `user:U...` 및 `channel:C...`
- Discord: `user:<id>` 및 `channel:<id>`
- Matrix (플러그인): `user:@user:server`, `room:!roomId:server`, 또는 `#alias:server`
- Microsoft Teams (플러그인): `user:<id>` 및 `conversation:<id>`
- Zalo (플러그인): 사용자 ID (Bot API)
- Zalo Personal / `zalouser` (플러그인): `zca`의 스레드 ID (DM/그룹) (`me`, `friend list`, `group list`)

## 자신 ("me")

```bash
openclaw directory self --channel zalouser
```

## 피어 (연락처/사용자)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## 그룹

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
