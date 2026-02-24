---
summary: "마터모스트(Mattermost) 봇 설정 및 OpenClaw 연동 가이드"
read_when:
  - 마터모스트를 OpenClaw에 연결하고 싶을 때
  - 봇 토큰(Bot Token) 및 웹소켓(WebSocket) 설정을 확인해야 할 때
title: "마터모스트"
---

# 마터모스트 (Mattermost / Plugin)

마터모스트는 자체 호스팅이 가능한 오픈 소스 팀 메신저 플랫폼입니다. OpenClaw는 플러그인을 통해 마터모스트 봇 토큰과 웹소켓(WebSocket)을 사용하여 연동됩니다.

## 중요: 플러그인 설치 필수
마터모스트 연동 기능은 별도의 플러그인으로 제공됩니다. 아래 명령어로 먼저 설치하세요:
```bash
openclaw plugins install @openclaw/mattermost
```

## 빠른 설정 방법

1.  **봇 계정 생성**: 마터모스트 관리자 대시보드에서 봇 계정을 생성하고 **봇 토큰(Bot Token)**을 복사합니다.
2.  **OpenClaw 설정 (`openclaw.json`)**:
    ```json5
    {
      "channels": {
        "mattermost": {
          "enabled": true,
          "botToken": "mm-토큰-값",
          "baseUrl": "https://chat.example.com", // 마터모스트 서버 주소
          "dmPolicy": "pairing"
        }
      }
    }
    ```
3.  **게이트웨이 실행**: `openclaw gateway`를 실행하여 연결을 완료합니다.

## 대화 모드 (`chatmode`)
채널에서의 응답 방식을 설정할 수 있습니다:
- `oncall` (기본값): 에이전트를 직접 멘션(@이름)할 때만 응답합니다.
- `onmessage`: 모든 메시지에 응답합니다.
- `onchar`: 메시지가 특정 기호(예: `!`, `>`)로 시작할 때 응답합니다.

## 주요 보안 및 기능
- **페어링**: 처음 DM을 보낼 때 페어링 코드로 승인해야 합니다.
- **반응(Reaction)**: 메시지에 이모지 리액션을 남길 수 있습니다.
- **채널 초대**: 에이전트를 특정 채널에 초대하여 함께 업무를 처리할 수 있습니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
