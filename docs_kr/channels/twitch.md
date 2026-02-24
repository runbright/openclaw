---
summary: "Twitch(트위치) 채팅 봇 설정 및 OpenClaw 연동 가이드"
read_when:
  - OpenClaw의 트위치 채팅 연동을 설정할 때
title: "Twitch"
---

# Twitch (플러그인)

OpenClaw는 IRC 연결을 통해 Twitch 채팅을 지원합니다. 에이전트가 트위치 사용자(봇 계정)로 접속하여 채널의 메시지를 수신하고 답변을 보낼 수 있습니다.

## 플러그인 설치

Twitch 연동을 위해서는 먼저 전용 플러그인을 설치해야 합니다:

```bash
openclaw plugins install @openclaw/twitch
```

## 빠른 설정 가이드 (초보자용)

1. **봇 계정 준비**: 봇으로 사용할 별도의 트위치 계정을 준비합니다.
2. **토큰 발급**: [Twitch Token Generator](https://twitchtokengenerator.com/)에서 토큰을 생성합니다.
   - **Bot Token** 항목을 선택합니다.
   - `chat:read` 및 `chat:write` 권한이 포함되어 있는지 확인합니다.
   - 발급된 **Client ID**와 **Access Token**을 복사합니다.
3. **사용자 ID 확인**: [ID 변환 도구](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)에서 자신의 트위치 닉네임을 입력해 숫자 형태의 **사용자 ID**를 확인합니다.
4. **OpenClaw 설정 (`openclaw.json`)**:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "봇_계정_닉네임",
      clientId: "발급받은_클라이언트_ID",
      accessToken: "oauth:발급받은_액세스_토큰",
      channel: "참여할_채널_닉네임",
      allowFrom: ["본인의_사용자_ID"], // 본인만 에이전트를 조작할 수 있도록 설정 권장
    },
  },
}
```

## 접근 제어 (Access Control)

트위치 채팅은 누구나 참여할 수 있으므로 보안 설정이 매우 중요합니다.

- **사용자 ID 허용 목록 (`allowFrom`)**: 닉네임은 변경될 수 있으므로 숫자 형태의 사용자 ID를 등록하는 것이 가장 안전합니다.
- **역할 기반 허용 (`allowedRoles`)**: `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"` 중 선택할 수 있습니다.
- **멘션 필수 (`requireMention`)**: 기본적으로 `true`로 설정되어 있어 에이전트를 직접 호출(@봇이름)할 때만 답변합니다.

## 주요 기능 및 특징

- **자동 답변**: 트위치 채팅창에서 에이전트와 실시간 대화가 가능합니다.
- **메시지 제한**: 한 메시지당 최대 500자까지 전송되며, 긴 메시지는 단어 단위로 자동 분할되어 전송됩니다.
- **토큰 갱신**: 자동 갱신 기능을 사용하려면 트위치 개발자 콘솔에서 직접 앱을 등록하고 `clientSecret`과 `refreshToken`을 입력해야 합니다.

## 트러블슈팅

- **봇이 응답하지 않음**: 사용자 ID가 `allowFrom`에 포함되어 있는지, 또는 `requireMention` 설정에 따라 봇을 멘션했는지 확인하세요.
- **인증 에러**: `accessToken`이 만료되었을 수 있습니다. 토큰을 새로 발급받아 업데이트하십시오.

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration)를 확인하세요.
---
