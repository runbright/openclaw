---
summary: "디스코드(Discord) 봇 연동 상태, 기능 및 설정 가이드"
read_when:
  - 디스코드 봇을 생성하고 서버에 추가하고 싶을 때
  - 디스코드 권한(Intents)이나 슬래시 명령어 설정을 확인해야 할 때
title: "디스코드"
---

# 디스코드 (Discord Bot API)

디스코드는 서버(길드), 채널, 그리고 개인 DM을 통해 OpenClaw와 대화할 수 있는 강력한 플랫폼입니다.

## 빠른 설정 방법

설정을 위해 디스코드 개발자 포털에서 봇을 생성하고 서버에 초대해야 합니다.

### 1단계: 디스코드 봇 생성
1.  [Discord Developer Portal](https://discord.com/developers/applications)에 접속하여 **New Application**을 클릭합니다.
2.  이름을 정하고 생성한 뒤, 왼쪽 메뉴에서 **Bot** 탭으로 이동합니다.
3.  **Reset Token** 버튼을 눌러 봇 토큰을 복사하여 안전한 곳에 저장해 둡니다.

### 2단계: 권한(Intents) 설정
**Bot** 페이지 중간의 **Privileged Gateway Intents** 섹션에서 다음 항목을 켭니다:
- **Message Content Intent** (필수)
- **Server Members Intent** (권장)

### 3단계: 서버 초대 URL 생성
1.  **OAuth2 → URL Generator** 메뉴로 이동합니다.
2.  **Scopes**에서 `bot`과 `applications.commands`를 체크합니다.
3.  아래에 나타나는 **Bot Permissions**에서 다음 권한을 체크합니다:
    - View Channels, Send Messages, Read Message History, Embed Links, Attach Files
4.  하단에 생성된 URL을 브라우저에 붙여넣어 자신의 서버에 봇을 초대합니다.

### 4단계: OpenClaw 설정
`openclaw.json` 파일에 복사한 토큰을 입력합니다.
```json5
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "복사한-봇-토큰"
    }
  }
}
```

### 5단계: 페어링 승인
봇에게 DM을 보내면 페어링 코드가 나타납니다. 서버 터미널에서 다음 명령어로 승인하세요:
```bash
openclaw pairing list discord
openclaw pairing approve discord <코드>
```

## 주요 기능
- **슬래시 명령어**: `/new`, `/reset` 등 디스코드 고유의 명령어를 사용할 수 있습니다.
- **파일 전송**: 에이전트가 생성한 이미지나 파일을 디스코드 채널에 직접 업로드합니다.
- **역할 기반 접근 제어**: 특정 역할을 가진 사용자만 에이전트와 대화하도록 설정할 수 있습니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
