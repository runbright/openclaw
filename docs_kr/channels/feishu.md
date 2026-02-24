---
summary: "비바(Feishu) 및 라크(Lark) 봇 연동 상태, 주요 기능 및 설정 가이드"
read_when:
  - 비바 또는 라크 앱을 OpenClaw에 연결하고 싶을 때
  - 앱 ID, 앱 시크릿 설정 및 권한(Permissions) 구성을 확인해야 할 때
title: "비바 (Feishu / Lark)"
---

# 비바 봇 (Feishu / Lark)

비바(Feishu) 및 라크(Lark) 채널은 팀 협업 플랫폼인 비바/라크를 AI 에이전트와 연동할 수 있게 해줍니다. 웹소켓(WebSocket) 기반의 롱 커넥션(Long connection)을 지원하므로 공인 IP나 웹훅 URL 노출 없이도 안전하게 메시지를 주고받을 수 있습니다.

## 중요: 플러그인 설치 필수
비바 연동 기능은 별도의 플러그인으로 제공됩니다. 아래 명령어로 먼저 설치하세요:
```bash
openclaw plugins install @openclaw/feishu
```

## 빠른 설정 방법

### 1단계: 비바 오픈 플랫폼 설정
1.  [비바 오픈 플랫폼](https://open.feishu.cn/app) (글로벌 사용자는 [Lark Open Platform](https://open.larksuite.com/app))에 접속하여 앱을 생성합니다.
2.  **인증 정보 복사**: 'Credentials & Basic Info'에서 **App ID**와 **App Secret**을 복사합니다.
3.  **권한 설정**: 'Permissions' 메뉴에서 메시지 읽기/쓰기(`im:message`, `im:message.p2p_msg:readonly` 등) 관련 권한을 부여합니다.
4.  **봇 활성화**: 'App Capability > Bot'에서 봇 기능을 활성화합니다.
5.  **이벤트 구독**: 'Event Subscription'에서 **Long Connection**을 선택하고 `im.message.receive_v1` 이벤트를 추가합니다.

### 2단계: OpenClaw 설정 (`openclaw.json`)
```json5
{
  "channels": {
    "feishu": {
      "enabled": true,
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx"
        }
      }
    }
  }
}
```

## 주요 기능
- **간편한 연결**: 웹소켓 방식을 사용하므로 방화벽 설정이나 포트 포워딩이 필요 없습니다.
- **실시간 응답 (Streaming)**: 인터랙티브 카드를 통해 AI가 답변을 생성하는 과정을 실시간으로 볼 수 있습니다.
- **다양한 미디어**: 텍스트뿐만 아니라 이미지, 파일, 오디오 메시지 수발신을 지원합니다.
- **페어링 시스템**: 처음 대화 시 페어링 코드로 승인하여 보안을 유지합니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
