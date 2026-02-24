---
summary: "`openclaw message` (전송 + 채널 액션) CLI 레퍼런스"
read_when:
  - 메시지 CLI 액션을 추가하거나 수정할 때
  - 아웃바운드 채널 동작을 변경할 때
title: "message"
---

# `openclaw message`

메시지 전송 및 채널 액션을 위한 단일 아웃바운드 명령어
(Discord/Google Chat/Slack/Mattermost (플러그인)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

## 사용법

```
openclaw message <subcommand> [flags]
```

채널 선택:

- `--channel`은 둘 이상의 채널이 구성된 경우 필수입니다.
- 정확히 하나의 채널만 구성된 경우, 해당 채널이 기본값이 됩니다.
- 값: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost는 플러그인 필요)

대상 형식 (`--target`):

- WhatsApp: E.164 또는 그룹 JID
- Telegram: chat id 또는 `@username`
- Discord: `channel:<id>` 또는 `user:<id>` (또는 `<@id>` 멘션; 숫자 ID는 채널로 처리)
- Google Chat: `spaces/<spaceId>` 또는 `users/<userId>`
- Slack: `channel:<id>` 또는 `user:<id>` (숫자 채널 ID 허용)
- Mattermost (플러그인): `channel:<id>`, `user:<id>`, 또는 `@username` (숫자 ID는 채널로 처리)
- Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>`, 또는 `username:<name>`/`u:<name>`
- iMessage: handle, `chat_id:<id>`, `chat_guid:<guid>`, 또는 `chat_identifier:<id>`
- MS Teams: conversation id (`19:...@thread.tacv2`) 또는 `conversation:<id>` 또는 `user:<aad-object-id>`

이름 조회:

- 지원되는 프로바이더(Discord/Slack 등)의 경우, `Help` 또는 `#help`와 같은 채널 이름은 디렉토리 캐시를 통해 확인됩니다.
- 캐시 미스 시, 프로바이더가 지원하는 경우 OpenClaw는 라이브 디렉토리 조회를 시도합니다.

## 공통 플래그

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (전송/폴/읽기 등의 대상 채널 또는 사용자)
- `--targets <name>` (반복; 브로드캐스트 전용)
- `--json`
- `--dry-run`
- `--verbose`

## 액션

### 핵심

- `send`
  - 채널: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (플러그인)/Signal/iMessage/MS Teams
  - 필수: `--target`, 그리고 `--message` 또는 `--media`
  - 선택: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Telegram 전용: `--buttons` (`channels.telegram.capabilities.inlineButtons` 허용 필요)
  - Telegram 전용: `--thread-id` (포럼 토픽 ID)
  - Slack 전용: `--thread-id` (스레드 타임스탬프; `--reply-to`는 동일 필드 사용)
  - WhatsApp 전용: `--gif-playback`

- `poll`
  - 채널: WhatsApp/Telegram/Discord/Matrix/MS Teams
  - 필수: `--target`, `--poll-question`, `--poll-option` (반복)
  - 선택: `--poll-multi`
  - Discord 전용: `--poll-duration-hours`, `--silent`, `--message`
  - Telegram 전용: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`

- `react`
  - 채널: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - 필수: `--message-id`, `--target`
  - 선택: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - 참고: `--remove`는 `--emoji`가 필요합니다 (`--emoji`를 생략하면 지원되는 곳에서 자신의 리액션을 지웁니다; /tools/reactions 참조)
  - WhatsApp 전용: `--participant`, `--from-me`
  - Signal 그룹 리액션: `--target-author` 또는 `--target-author-uuid` 필수

- `reactions`
  - 채널: Discord/Google Chat/Slack
  - 필수: `--message-id`, `--target`
  - 선택: `--limit`

- `read`
  - 채널: Discord/Slack
  - 필수: `--target`
  - 선택: `--limit`, `--before`, `--after`
  - Discord 전용: `--around`

- `edit`
  - 채널: Discord/Slack
  - 필수: `--message-id`, `--message`, `--target`

- `delete`
  - 채널: Discord/Slack/Telegram
  - 필수: `--message-id`, `--target`

- `pin` / `unpin`
  - 채널: Discord/Slack
  - 필수: `--message-id`, `--target`

- `pins` (목록)
  - 채널: Discord/Slack
  - 필수: `--target`

- `permissions`
  - 채널: Discord
  - 필수: `--target`

- `search`
  - 채널: Discord
  - 필수: `--guild-id`, `--query`
  - 선택: `--channel-id`, `--channel-ids` (반복), `--author-id`, `--author-ids` (반복), `--limit`

### 스레드

- `thread create`
  - 채널: Discord
  - 필수: `--thread-name`, `--target` (채널 ID)
  - 선택: `--message-id`, `--message`, `--auto-archive-min`

- `thread list`
  - 채널: Discord
  - 필수: `--guild-id`
  - 선택: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - 채널: Discord
  - 필수: `--target` (스레드 ID), `--message`
  - 선택: `--media`, `--reply-to`

### 이모지

- `emoji list`
  - Discord: `--guild-id`
  - Slack: 추가 플래그 없음

- `emoji upload`
  - 채널: Discord
  - 필수: `--guild-id`, `--emoji-name`, `--media`
  - 선택: `--role-ids` (반복)

### 스티커

- `sticker send`
  - 채널: Discord
  - 필수: `--target`, `--sticker-id` (반복)
  - 선택: `--message`

- `sticker upload`
  - 채널: Discord
  - 필수: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### 역할 / 채널 / 멤버 / 음성

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (Discord의 경우 `--guild-id` 추가)
- `voice status` (Discord): `--guild-id`, `--user-id`

### 이벤트

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - 선택: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### 관리 (Discord)

- `timeout`: `--guild-id`, `--user-id` (선택 `--duration-min` 또는 `--until`; 둘 다 생략하면 타임아웃 해제)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout`도 `--reason`을 지원합니다

### 브로드캐스트

- `broadcast`
  - 채널: 구성된 모든 채널; 모든 프로바이더를 대상으로 하려면 `--channel all` 사용
  - 필수: `--targets` (반복)
  - 선택: `--message`, `--media`, `--dry-run`

## 예시

Discord 답장 전송:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

컴포넌트가 포함된 Discord 메시지 전송:

```
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

전체 스키마는 [Discord 컴포넌트](/channels/discord#interactive-components)를 참조하세요.

Discord 투표 생성:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Telegram 투표 생성 (2분 후 자동 종료):

```
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

Teams 프로액티브 메시지 전송:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Teams 투표 생성:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Slack에서 리액션:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Signal 그룹에서 리액션:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Telegram 인라인 버튼 전송:

```
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```
