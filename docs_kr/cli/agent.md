---
summary: "`openclaw agent` CLI 참조 (Gateway를 통해 에이전트 턴 하나 실행)"
read_when:
  - 스크립트에서 에이전트 턴 하나를 실행하려는 경우 (선택적으로 응답 전달)
title: "agent"
---

# `openclaw agent`

Gateway를 통해 에이전트 턴을 실행합니다 (임베디드 모드는 `--local` 사용).
`--agent <id>`를 사용하여 구성된 에이전트를 직접 대상으로 지정합니다.

관련 항목:

- 에이전트 전송 도구: [Agent send](/tools/agent-send)

## 예시

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
