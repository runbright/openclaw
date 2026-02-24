---
summary: "텔레그램(Telegram) 봇 연동 상태, 기능 및 설정 가이드"
read_when:
  - 텔레그램 봇을 생성하고 OpenClaw에 연결하고 싶을 때
  - 텔레그램 그룹 대화나 권한 설정(Privacy Mode)을 변경할 때
title: "텔레그램"
---

# 텔레그램 (Telegram Bot API)

텔레그램은 OpenClaw를 처음 시작할 때 가장 권장하는 채널입니다. 설정이 매우 빠르고 안정적이며, 개인 대화뿐만 아니라 그룹 대화도 지원합니다.

## 빠른 설정 방법

1.  **봇 생성**: 텔레그램에서 **@BotFather**를 찾아 대화를 시작합니다. `/newbot` 명령어를 입력하고 안내에 따라 봇 이름과 아이디를 만들면 **API 토큰(Token)**을 줍니다.
2.  **토큰 설정**: `openclaw.json` 파일에 토큰을 입력합니다.
    ```json5
    {
      "channels": {
        "telegram": {
          "enabled": true,
          "botToken": "복사한-토큰-값",
          "dmPolicy": "pairing" // 보안을 위해 처음엔 페어링 방식을 권장합니다.
        }
      }
    }
    ```
3.  **게이트웨이 실행**: `openclaw gateway`를 실행합니다.
4.  **페어링 승인**: 텔레그램 봇에게 아무 메시지나 보낸 뒤, 서버 터미널에서 다음 명령어를 실행하여 승인합니다.
    ```bash
    openclaw pairing list telegram
    openclaw pairing approve telegram <코드>
    ```

## 그룹 대화 설정

봇을 그룹에 초대하여 사용할 수 있습니다.

- **프라이버시 모드 (Privacy Mode)**: 기본적으로 봇은 자신을 멘션(`@봇이름`)한 메시지만 볼 수 있습니다. 모든 메시지를 보게 하려면 @BotFather에게 `/setprivacy` 명령어로 설정을 끄거나, 봇을 그룹 관리자로 지정해야 합니다.
- **그룹 허용 목록**: 특정 그룹에서만 봇이 작동하게 하려면 `channels.telegram.groups` 설정에서 그룹 ID를 지정하세요.

## 주요 보안 설정 (dmPolicy)

- `pairing` (기본값): 처음 대화할 때 페어링 코드로 승인받은 사용자만 이용 가능합니다.
- `allowlist`: 지정된 사용자 ID(숫자)만 이용 가능합니다.
- `open`: 누구나 봇과 대화할 수 있습니다. (추천하지 않음)

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
