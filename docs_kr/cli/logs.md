---
summary: "`openclaw logs` CLI 참조 (RPC를 통해 Gateway 로그 추적)"
read_when:
  - SSH 없이 원격으로 Gateway 로그를 추적해야 하는 경우
  - 도구용 JSON 로그 라인이 필요한 경우
title: "logs"
---

# `openclaw logs`

RPC를 통해 Gateway 파일 로그를 추적합니다 (원격 모드에서 작동).

관련 항목:

- 로깅 개요: [Logging](/logging)

## 예시

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

`--local-time`을 사용하면 타임스탬프를 로컬 시간대로 렌더링합니다.
