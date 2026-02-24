---
summary: "슬랙(Slack) 앱 연동, 소켓 모드(Socket Mode) 및 HTTP 이벤트 설정 가이드"
read_when:
  - 슬랙 워크스페이스에 OpenClaw를 연결하고 싶을 때
  - 슬랙 앱 토큰(App Token)이나 봇 토큰(Bot Token) 설정을 확인해야 할 때
title: "슬랙"
---

# 슬랙 (Slack)

슬랙 채널은 슬랙 앱 인티그레이션을 통해 에이전트와 대화할 수 있게 해줍니다. 기본적으로 방화벽 설정이 필요 없는 **소켓 모드(Socket Mode)**를 권장합니다.

## 빠른 설정 방법 (소켓 모드)

1.  **슬랙 앱 생성**: [Slack API](https://api.slack.com/apps) 사이트에서 새로운 앱을 생성합니다.
2.  **소켓 모드 활성화**: 앱 설정에서 **Socket Mode**를 켭니다. 이때 대화형 기능을 위한 **App Token**(`xapp-...`)이 생성됩니다.
3.  **봇 토큰 발급**: 앱을 워크스페이스에 설치하면 **Bot User OAuth Token**(`xoxb-...`)이 발급됩니다.
4.  **OpenClaw 설정**: `openclaw.json` 파일에 두 토큰을 입력합니다.
    ```json5
    {
      "channels": {
        "slack": {
          "enabled": true,
          "mode": "socket", // 기본값
          "appToken": "xapp-...",
          "botToken": "xoxb-..."
        }
      }
    }
    ```
5.  **이벤트 구독**: **Event Subscriptions** 메뉴에서 다음 이벤트들을 추가하고 저장합니다:
    - `app_mention`, `message.channels`, `message.im`, `message.groups` 등
6.  **게이트웨이 실행**: `openclaw gateway`를 실행하여 연결을 완료합니다.

## 접근 제어 (DM 정책)

- `pairing` (기본값): 처음 DM을 보낼 때 페어링 승인이 필요합니다.
- `allowlist`: 허용된 슬랙 사용자 ID 목록만 대화가 가능합니다.
- `open`: 워크스페이스 내 누구나 대화할 수 있습니다.

## 주요 기능
- **슬래시 명령어**: `/new`, `/reset` 등 슬랙 고유의 명령어를 지원합니다.
- **에이전트 이름 설정**: `chat:write.customize` 권한이 있는 경우, 에이전트가 본인의 이름과 아이콘으로 메시지를 보냅니다.
- **워크스페이스 채널**: 개별 프로젝트 채널에 에이전트를 초대하여 협업할 수 있습니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
