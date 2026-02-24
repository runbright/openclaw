---
summary: "시놀로지 챗(Synology Chat) 웹훅 설정 및 OpenClaw 연동 가이드"
read_when:
  - 시놀로지 NAS의 챗 기능을 OpenClaw와 연결하고 싶을 때
  - 인커밍/아웃고잉 웹훅 설정을 확인해야 할 때
title: "시놀로지 챗"
---

# 시놀로지 챗 (Synology Chat / Plugin)

시놀로지 챗 채널은 시놀로지 NAS 사용자들을 위한 메시징 서비스와 OpenClaw를 연결해줍니다. 인커밍(Incoming) 및 아웃고잉(Outgoing) 웹훅을 통해 메시지를 주고받습니다.

## 중요: 플러그인 설치 필수
시놀로지 챗 연동 기능은 별도의 플러그인으로 제공됩니다. 아래 명령어로 먼저 설치하세요:
```bash
openclaw plugins install ./extensions/synology-chat
```

## 빠른 설정 방법

1.  **시놀로지 챗 통합 설정**:
    - **들어오는 웹훅(Incoming Webhook)**: 생성 후 URL을 복사합니다 (에이전트가 답변을 보낼 때 사용).
    - **나가는 웹훅(Outgoing Webhook)**: 생성 후 토큰(Token)을 복사하고, 대상 URL을 OpenClaw 서버 주소(예: `https://서버주소/webhook/synology`)로 설정합니다.
2.  **OpenClaw 설정 (`openclaw.json`)**:
    ```json5
    {
      "channels": {
        "synology-chat": {
          "enabled": true,
          "token": "아웃고잉-웹훅-토큰",
          "incomingUrl": "복사한-인커밍-웹훅-URL",
          "webhookPath": "/webhook/synology",
          "dmPolicy": "allowlist",
          "allowedUserIds": ["123456"] // 허용할 시놀로지 사용자 ID
        }
      }
    }
    ```
3.  **게이트웨이 실행**: `openclaw gateway`를 실행하여 연동을 시작합니다.

## 주요 기능 및 보안
- **사용자 관리**: 보안을 위해 `allowedUserIds`에 지정된 사용자만 에이전트와 대화하도록 설정하는 것을 권장합니다.
- **메시지 전송**: `openclaw message send` 명령어를 통해 특정 사용자 ID로 직접 메시지를 보낼 수 있습니다.
- **이미지 및 파일**: URL 기반의 파일 전송 기능을 지원합니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
