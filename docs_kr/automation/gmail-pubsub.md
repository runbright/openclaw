---
summary: "Gmail Pub/Sub을 gogcli를 통해 OpenClaw 웹훅에 연결하여 이메일 알림 및 처리 자동화"
read_when:
  - Gmail 받은 편지함 알림을 OpenClaw 에이전트에 연결하고 싶을 때
  - 이메일 도착 시 에이전트가 자동으로 요약하거나 답변하게 하고 싶을 때
title: "Gmail PubSub"
---

# Gmail Pub/Sub 연결 가이드

이 가이드는 Gmail에 새 메일이 도착하면 구글 클라우드의 **Pub/Sub** 기능을 통해 **OpenClaw** 에이전트에 알림을 보내고, 에이전트가 메일 내용을 요약하거나 처리하도록 설정하는 방법을 설명합니다.

## 필수 준비물
- 구글 클라우드 SDK (`gcloud`) 설치
- `gogcli` 설치 및 인증 (Gmail 계정 접근용)
- OpenClaw 웹훅 활성화 (참고: [Webhooks](/automation/webhook))
- **Tailscale**: 외부에서 알림을 받기 위한 안전한 HTTPS 통로(Funnel)로 권장됩니다.

## 1단계: OpenClaw 설정
`openclaw.json` 파일에서 Gmail 프리셋을 활성화합니다.

```json5
{
  "hooks": {
    "enabled": true,
    "presets": ["gmail"],
    "mappings": [
      {
        "match": { "path": "gmail" },
        "action": "agent",
        "messageTemplate": "새 이메일 도착 - 보낸사람: {{messages[0].from}}\n제목: {{messages[0].subject}}\n내용: {{messages[0].snippet}}",
        "deliver": true,
        "channel": "last" // 마지막으로 대화한 채널(예: WhatsApp)로 알림 전송
      }
    ]
  }
}
```

## 2단계: 자동 설정 마법사 (추천)
복잡한 클라우드 설정을 자동으로 처리해 주는 도우미를 실행하세요:

```bash
openclaw webhooks gmail setup --account 내이메일@gmail.com
```

**이 마법사가 하는 일:**
- 필요한 패키지(gcloud, gogcli, tailscale)를 확인합니다.
- Tailscale Funnel을 통해 외부에서 접속 가능한 URL을 만듭니다.
- Gmail 감시(Watch) 스크립트를 백그라운드 서비스로 등록합니다.

## 3단계: 수동 실행 (디버깅용)
설정이 잘 되었는지 확인하기 위해 수동으로 감시 서비스를 시작할 수 있습니다:

```bash
openclaw webhooks gmail run
```

## 작동 원리
1.  **Gmail**: 메일이 도착하면 구글 클라우드 Pub/Sub에 알립니다.
2.  **Tailscale Funnel**: 구글의 알림을 내 로컬 컴퓨터의 OpenClaw로 안전하게 전달합니다.
3.  **OpenClaw**: 알림을 받으면 `gog` 도구를 사용해 실제 메일 내용을 가져온 뒤, 설정된 에이전트(AI)에게 전달합니다.

## 문제 해결
- **알림이 오지 않음**: `tailscale serve status` 명령어로 외부 URL이 정상인지 확인하세요.
- **권한 오류**: 구글 클라우드에서 Pub/Sub 주제(Topic)에 대한 권한 설정이 올바른지 확인하세요.

---
상세한 웹훅 설정은 [웹훅 가이드](/automation/webhook)를 참고하세요.
---
