---
summary: "`openclaw status` (진단, 프로브, 사용량 스냅샷) CLI 레퍼런스"
read_when:
  - 채널 상태 + 최근 세션 수신자의 빠른 진단이 필요할 때
  - 디버깅을 위한 복사 가능한 "all" 상태가 필요할 때
title: "status"
---

# `openclaw status`

채널 + 세션 진단.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

참고:

- `--deep`은 라이브 프로브를 실행합니다 (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
- 여러 에이전트가 구성된 경우 에이전트별 세션 스토어를 출력에 포함합니다.
- 개요에는 사용 가능한 경우 Gateway + 노드 호스트 서비스 설치/런타임 상태가 포함됩니다.
- 개요에는 업데이트 채널 + git SHA (소스 체크아웃의 경우)가 포함됩니다.
- 업데이트 정보는 개요에 표시됩니다; 업데이트가 사용 가능한 경우, 상태는 `openclaw update` 실행 힌트를 출력합니다 ([업데이트](/install/updating) 참조).
