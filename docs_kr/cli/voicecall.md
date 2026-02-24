---
summary: "`openclaw voicecall` (음성 통화 플러그인 명령 표면) CLI 레퍼런스"
read_when:
  - 음성 통화 플러그인을 사용하며 CLI 진입점이 필요할 때
  - `voicecall call|continue|status|tail|expose`의 빠른 예시가 필요할 때
title: "voicecall"
---

# `openclaw voicecall`

`voicecall`은 플러그인 제공 명령어입니다. 음성 통화 플러그인이 설치되고 활성화된 경우에만 나타납니다.

주요 문서:

- 음성 통화 플러그인: [음성 통화](/plugins/voice-call)

## 일반 명령어

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## 웹훅 노출 (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

보안 참고: 신뢰하는 네트워크에만 웹훅 엔드포인트를 노출하세요. 가능한 경우 Funnel보다 Tailscale Serve를 권장합니다.
