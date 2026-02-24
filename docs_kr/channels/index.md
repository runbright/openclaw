---
summary: "OpenClaw가 연결할 수 있는 다양한 대화 채널(메신저) 안내"
read_when:
  - OpenClaw를 어떤 메신저에서 사용할지 결정하고 싶을 때
  - 지원되는 플랫폼 목록을 확인하고 싶을 때
title: "대화 채널"
---

# 대화 채널 (Chat Channels)

OpenClaw는 사용자가 이미 사용 중인 거의 모든 메신저 앱을 통해 대화할 수 있습니다. 텍스트 대화는 모든 채널에서 기본적으로 지원되며, 이미지/파일 전송이나 리액션 등은 채널별로 차이가 있을 수 있습니다.

## 지원되는 채널 목록

- [WhatsApp](/channels/whatsapp) — Baileys 라이브러리를 사용하며 QR 코드로 페어링합니다.
- [Telegram](/channels/telegram) — 가장 설정이 쉬운 채널입니다. 봇 토큰(Bot Token)만 있으면 바로 시작할 수 있습니다.
- [Discord](/channels/discord) — 디스코드 서버, 채널, DM에서 대화할 수 있습니다.
- [Slack](/channels/slack) — 워크스페이스용 앱을 통해 연동됩니다.
- [iMessage (BlueBubbles)](/channels/bluebubbles) — 아이메시지 사용을 위한 권장 방법입니다.
- [Signal](/channels/signal) — 강력한 보안과 개인정보 보호를 중시할 때 사용합니다.
- [Microsoft Teams](/channels/msteams) — 기업용 메신저 연동을 지원합니다 (플러그인 필요).
- [WebChat](/web/webchat) — 웹 브라우저에서 직접 에이전트와 대화하는 대시보드 UI입니다.

...그 외에도 IRC, LINE, Matrix, Nostr 등 다양한 채널을 지원합니다.

## 안내 사항

- **동시 사용 가능**: 여러 채널을 동시에 연동해두고, 상황에 맞춰 에이전트와 대화할 수 있습니다.
- **간편한 설정**: 처음 시작한다면 **Telegram**이 가장 빠르고 간편합니다.
- **보안**: 알지 못하는 사람의 접근을 막기 위해 페어링 코드나 허용 목록(Allowlist) 기능을 제공합니다.
- **그룹 대화**: 채널별로 그룹 대화 지원 방식이 다릅니다. 자세한 내용은 [그룹 가이드](/channels/groups)를 참고하세요.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
