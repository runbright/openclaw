---
summary: "마이크로소프트 팀즈(Microsoft Teams) 연동 상태, 플러그인 설치 및 구성 가이드"
read_when:
  - MS 팀즈를 OpenClaw에 연결하고 싶을 때
  - 애저 봇(Azure Bot) 설정 및 권한(Graph API) 구성을 확인해야 할 때
title: "마이크로소프트 팀즈"
---

# 마이크로소프트 팀즈 (Microsoft Teams / Plugin)

마이크로소프트 팀즈 채널은 전용 플러그인을 통해 연동됩니다. 텍스트 대화, 파일 전송, 설문 조사(Adaptive Cards 사용) 등을 지원합니다.

## 중요: 플러그인 설치 필수
MS 팀즈 연동 기능은 핵심 설치본에 포함되어 있지 않습니다. 아래 명령어로 먼저 플러그인을 설치해야 합니다:
```bash
openclaw plugins install @openclaw/msteams
```

## 빠른 설정 방법 (초보자용)

1.  **애저 봇(Azure Bot) 생성**: [Azure Portal](https://portal.azure.com/)에서 '애저 봇'을 생성하고 다음 정보를 복사합니다:
    - App ID
    - Client Secret (App Password)
    - Tenant ID
2.  **공개 엔드포인트 설정**: 게이트웨이가 외부에서 접속 가능하도록 설정하고, `/api/messages` 경로를 봇의 'Messaging endpoint'로 지정합니다.
3.  **OpenClaw 설정 (`openclaw.json`)**:
    ```json5
    {
      "channels": {
        "msteams": {
          "enabled": true,
          "appId": "<APP_ID>",
          "appPassword": "<APP_PASSWORD>",
          "tenantId": "<TENANT_ID>",
          "webhook": { "port": 3978, "path": "/api/messages" }
        }
      }
    }
    ```
4.  **팀즈 앱 패키지 설치**: 생성한 봇을 포함하는 앱 패키지를 만들어서 팀즈에 업로드합니다.

## 작동 방식 및 기능
- **DM 및 그룹**: 1:1 대화뿐만 아니라 그룹 채팅, 팀 채널에서의 대화도 지원합니다.
- **보안**: 기본적으로 `pairing` 모드가 적용됩니다. 처음 대화 시 페어링 코드로 승인해야 합니다.
- **파일 전송**: 그룹 대화에서 파일을 주고받으려면 별도의 SharePoint 권한(Graph API) 설정이 필요할 수 있습니다.
- **설문 조사**: 팀즈 고유의 '어댑티브 카드(Adaptive Card)'를 사용하여 투표 기능을 제공합니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
