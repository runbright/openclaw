---
summary: "채널별 주요 장애 증상 및 빠른 해결 방법 가이드"
read_when:
  - 채널이 연결된 것 같으나 에이전트의 응답이 없을 때
  - 특정 메신저(WhatsApp, Telegram 등) 관련 문제를 해결해야 할 때
title: "채널 문제 해결"
---

# 채널 문제 해결 (Channel Troubleshooting)

채널은 연결된 상태(`Connected`)인데 동작이 이상할 때 아래 진단 단계와 해결 방법을 참고하세요.

## 진단 명령어 (Command Ladder)

문제가 발생하면 다음 명령어들을 순서대로 실행해 보세요:

```bash
openclaw status                # 전체 시스템 상태 확인
openclaw gateway status        # 게이트웨이 및 RPC 통신 확인
openclaw logs --follow         # 실시간 오류 로그 확인
openclaw doctor                # 시스템 설정 및 파일 무결성 진단
openclaw channels status --probe # 채널별 상세 연결 상태 확인
```

## 주요 채널별 해결 방법

### 1. 왓츠앱 (WhatsApp)
- **증상**: 연결됨 상태인데 내 메시지에 답장이 없음
    - **해결**: `openclaw pairing list whatsapp` 명령어로 승인 대기 중인 사용자가 있는지 확인하고 승인하세요.
- **증상**: 그룹 메시지를 무시함
    - **해결**: 봇을 언급(`@봇이름`)했는지, 혹은 설정에서 `requireMention` 옵션이 켜져 있는지 확인하세요.

### 2. 텔레그램 (Telegram)
- **증상**: `/start`를 눌렀는데 아무 반응이 없음
    - **해결**: `openclaw pairing list telegram`에서 페어링 코드로 승인을 완료했는지 확인하세요.
- **증상**: 그룹에서 봇이 조용함
    - **해결**: @BotFather를 통해 봇의 'Privacy Mode'를 끄거나, 봇을 그룹 관리자로 지정해야 메시지를 읽을 수 있습니다.

### 3. 디스코드 (Discord)
- **증상**: 봇이 온라인인데 채널 메시지에 답이 없음
    - **해결**: 디스코드 개발자 포털에서 'Message Content Intent' 권한이 켜져 있는지 확인하세요.
- **증상**: DM 답장이 안 옴
    - **해결**: 서버 설정에서 '서버 멤버의 DM 허용'이 켜져 있는지 확인하고 페어링을 진행하세요.

### 4. 슬랙 (Slack)
- **증상**: 소켓 모드 연결은 되었는데 응답이 없음
    - **해결**: 앱 토큰(`xapp-`)과 봇 토큰(`xoxb-`)이 정확한지, 필요한 이벤트 구독(app_mention 등)이 되어 있는지 확인하세요.

---
여전히 문제가 해결되지 않는다면 [게이트웨이 문제 해결](/gateway/troubleshooting) 문서를 확인해 보세요.
---
