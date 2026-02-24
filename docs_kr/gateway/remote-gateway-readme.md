---
summary: "OpenClaw.app을 원격 게이트웨이에 연결하기 위한 SSH 터널 설정 가이드"
read_when: "macOS 앱을 SSH를 통해 원격 게이트웨이에 연결할 때"
title: "원격 게이트웨이 설정"
---

# 원격 게이트웨이와 macOS 앱 연결

OpenClaw.app(macOS 앱)은 SSH 터널링을 사용하여 원격지에 있는 게이트웨이에 접속할 수 있습니다. 이 가이드는 그 설정 방법을 설명합니다.

## 작동 원리

클라이언트 PC의 OpenClaw.app이 로컬 포트(`127.0.0.1:18789`)로 접속하면, SSH 터널이 이 패킷을 원격 서버의 게이트웨이 포트로 전달합니다.

## 빠른 설정 단계

### 1단계: SSH 설정 추가
`~/.ssh/config` 파일을 열고 다음 내용을 추가합니다:

```ssh
Host remote-gateway
    HostName <원격_IP_주소>       # 예: 1.23.45.67
    User <원격_사용자_이름>      # 예: ubuntu
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa   # 본인의 SSH 키 경로
```

### 2단계: SSH 키 복사
비밀번호 입력 없이 접속할 수 있도록 공개키를 원격지로 복사합니다:
```bash
ssh-copy-id -i ~/.ssh/id_rsa <원격_사용자>@<원격_IP>
```

### 3단계: 게이트웨이 토큰 설정
클라이언트 PC에서 게이트웨이 인증 토큰을 환경 변수로 설정합니다:
```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "여러분의_토큰"
```

### 4단계: SSH 터널 실행
```bash
ssh -N remote-gateway &
```

### 5단계: OpenClaw.app 재시작
OpenClaw.app을 완전히 종료(Command + Q)한 후 다시 실행하면 자동으로 원격 게이트웨이에 연결됩니다.

---

## 로그인 시 자동 실행 설정 (Launch Agent)

맥을 켤 때마다 터널이 자동으로 실행되도록 설정할 수 있습니다.

1. **설정 파일 생성**: `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist` 경로에 파일을 만들고 아래 내용을 복사해 넣습니다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

2. **서비스 등록**:
```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

이제 터널은 로그인 시 자동 시작되며, 중단될 경우 자동으로 재시작됩니다.

---

## 문제 해결 요령

- **터널 실행 확인**: `ps aux | grep "ssh -N remote-gateway"` 또는 `lsof -i :18789` 명령어로 포트가 열려 있는지 확인하세요.
- **터널 재시작**: `launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel`
- **터널 중지**: `launchctl bootout gui/$UID/bot.molt.ssh-tunnel`

상세한 보안 정책이나 기술적 배경은 [영문 원본 문서](/gateway/remote-gateway-readme)를 참고하십시오.
---
