---
summary: "IRC 플러그인 설정, 접속 관리 및 보안 대화 가이드"
read_when:
  - irc 채널(#room) 또는 DM을 OpenClaw에 연결하고 싶을 때
  - 닉네임 서버(NickServ) 설정 또는 사용자 허용 목록을 관리해야 할 때
title: "IRC"
---

# IRC (Internet Relay Chat)

OpenClaw는 IRC 채널(#방이름)과 개인 메시지(DM) 연동을 지원합니다. 고전적인 채팅 환경에서 에이전트와 대화하고 싶을 때 유용합니다.

## 빠른 설정 방법

1.  `openclaw.json` 파일에 IRC 설정을 추가합니다.
2.  최소한 아래 항목들은 설정해야 합니다:
    ```json5
    {
      "channels": {
        "irc": {
          "enabled": true,
          "host": "irc.libera.chat",
          "port": 6697,
          "tls": true, // 보안 연결 권장
          "nick": "my-ai-bot",
          "channels": ["#my-channel"] // 입장할 채널 목록
        }
      }
    }
    ```
3.  게이트웨이를 실행합니다: `openclaw gateway run`

## 보안 및 접근 제어
- **페어링**: 개인 메시지는 기본적으로 `pairing` 모드가 적용됩니다.
- **채널 보안**: 기본적으로 모든 채널 대화는 차단되어 있거나 멘션(`my-ai-bot: 안녕`) 시에만 응답합니다.
- **아이덴티티**: 특정 사용자만 봇을 쓰게 하려면 `nick!user@host` 형태의 전체 식별자를 허용 목록(`allowFrom`)에 추가하는 것이 안전합니다.

## 주요 기능
- **NickServ 지원**: 계정 비밀번호를 설정하여 닉네임 소유권을 인증할 수 있습니다.
- **멘션 제어**: `requireMention: false` 설정을 통해 멘션 없이도 봇이 모든 메시지에 반응하게 할 수 있습니다.
- **도구 제한**: 공개 채널에서는 보안을 위해 에이전트가 사용할 수 있는 도구(파일 쓰기 등)를 제한하는 것이 좋습니다.

## 문제 해결
- **응답이 없을 때**: 봇이 채널에 들어와 있는지 확인하고, 로그(`openclaw logs --follow`)에서 'missing-mention' 메시지가 찍히는지 확인하세요. 멘션 없이 답하게 하려면 `requireMention: false`를 추가하세요.

---
구체적인 보안 정책은 [보안 가이드](/gateway/security)를 참고하세요.
---
