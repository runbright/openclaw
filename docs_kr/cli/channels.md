---
summary: "`openclaw channels` CLI 참조 (계정, 상태, 로그인/로그아웃, 로그)"
read_when:
  - 채널 계정을 추가/제거하려는 경우 (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (플러그인)/Signal/iMessage)
  - 채널 상태를 확인하거나 채널 로그를 추적하려는 경우
title: "channels"
---

# `openclaw channels`

채팅 채널 계정과 Gateway에서의 런타임 상태를 관리합니다.

관련 문서:

- 채널 가이드: [Channels](/channels/index)
- Gateway 구성: [Configuration](/gateway/configuration)

## 일반 명령

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## 계정 추가 / 제거

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

팁: `openclaw channels add --help`로 채널별 플래그를 확인할 수 있습니다 (토큰, 앱 토큰, signal-cli 경로 등).

## 로그인 / 로그아웃 (대화형)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## 문제 해결

- 광범위한 프로브를 위해 `openclaw status --deep`를 실행합니다.
- 안내에 따른 수정을 위해 `openclaw doctor`를 사용합니다.
- `openclaw channels list`에서 `Claude: HTTP 403 ... user:profile`이 출력되면 사용량 스냅샷에 `user:profile` 범위가 필요합니다. `--no-usage`를 사용하거나, claude.ai 세션 키(`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`)를 제공하거나, Claude Code CLI를 통해 재인증합니다.

## 기능 프로브

프로바이더 기능 힌트(인텐트/범위, 가능한 경우)와 정적 기능 지원을 가져옵니다:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

참고사항:

- `--channel`은 선택 사항입니다. 생략하면 확장 프로그램을 포함한 모든 채널이 나열됩니다.
- `--target`은 `channel:<id>` 또는 숫자 채널 ID를 받으며 Discord에만 적용됩니다.
- 프로브는 프로바이더별로 다릅니다: Discord 인텐트 + 선택적 채널 권한; Slack 봇 + 사용자 범위; Telegram 봇 플래그 + 웹훅; Signal 데몬 버전; MS Teams 앱 토큰 + Graph 역할/범위(알려진 경우 주석 표시). 프로브가 없는 채널은 `Probe: unavailable`로 보고합니다.

## 이름을 ID로 변환

프로바이더 디렉토리를 사용하여 채널/사용자 이름을 ID로 변환합니다:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

참고사항:

- `--kind user|group|auto`를 사용하여 대상 유형을 강제할 수 있습니다.
- 동일한 이름을 가진 항목이 여러 개 있는 경우 활성 매치가 우선됩니다.
