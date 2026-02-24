---
summary: "`openclaw sessions` (저장된 세션 목록 + 사용량) CLI 레퍼런스"
read_when:
  - 저장된 세션을 나열하고 최근 활동을 확인하고 싶을 때
title: "sessions"
---

# `openclaw sessions`

저장된 대화 세션을 나열합니다.

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

범위 선택:

- 기본값: 구성된 기본 에이전트 스토어
- `--agent <id>`: 하나의 구성된 에이전트 스토어
- `--all-agents`: 모든 구성된 에이전트 스토어 집계
- `--store <path>`: 명시적 스토어 경로 (`--agent` 또는 `--all-agents`와 결합 불가)

JSON 예시:

`openclaw sessions --all-agents --json`:

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## 정리 유지보수

다음 쓰기 주기를 기다리지 않고 지금 유지보수를 실행합니다:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup`은 구성의 `session.maintenance` 설정을 사용합니다:

- 범위 참고: `openclaw sessions cleanup`은 세션 스토어/트랜스크립트만 유지보수합니다. cron 실행 로그(`cron/runs/<jobId>.jsonl`)는 정리하지 않으며, 이는 [Cron 구성](/automation/cron-jobs#configuration)의 `cron.runLog.maxBytes` 및 `cron.runLog.keepLines`로 관리되고 [Cron 유지보수](/automation/cron-jobs#maintenance)에서 설명됩니다.

- `--dry-run`: 쓰기 없이 정리/제한될 항목 수를 미리 봅니다.
  - 텍스트 모드에서 dry-run은 세션별 작업 테이블(`Action`, `Key`, `Age`, `Model`, `Flags`)을 출력하여 유지될 것과 제거될 것을 확인할 수 있습니다.
- `--enforce`: `session.maintenance.mode`가 `warn`인 경우에도 유지보수를 적용합니다.
- `--active-key <key>`: 디스크 예산 제거에서 특정 활성 키를 보호합니다.
- `--agent <id>`: 하나의 구성된 에이전트 스토어에 대해 정리를 실행합니다.
- `--all-agents`: 모든 구성된 에이전트 스토어에 대해 정리를 실행합니다.
- `--store <path>`: 특정 `sessions.json` 파일에 대해 실행합니다.
- `--json`: JSON 요약을 출력합니다. `--all-agents`와 함께 사용 시, 스토어당 하나의 요약이 출력됩니다.

`openclaw sessions cleanup --all-agents --dry-run --json`:

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

관련 항목:

- 세션 구성: [구성 레퍼런스](/gateway/configuration-reference#session)
