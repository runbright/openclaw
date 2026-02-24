---
summary: "`openclaw health` CLI 참조 (RPC를 통한 Gateway 상태 엔드포인트)"
read_when:
  - 실행 중인 Gateway의 상태를 빠르게 확인하려는 경우
title: "health"
---

# `openclaw health`

실행 중인 Gateway에서 상태를 가져옵니다.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

참고사항:

- `--verbose`는 실시간 프로브를 실행하고 여러 계정이 구성된 경우 계정별 소요 시간을 출력합니다.
- 출력에는 여러 에이전트가 구성된 경우 에이전트별 세션 저장소가 포함됩니다.
