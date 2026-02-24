---
summary: "`openclaw agents` CLI 참조 (목록/추가/삭제/아이덴티티 설정)"
read_when:
  - 여러 격리된 에이전트가 필요한 경우 (워크스페이스 + 라우팅 + 인증)
title: "agents"
---

# `openclaw agents`

격리된 에이전트를 관리합니다 (워크스페이스 + 인증 + 라우팅).

관련 항목:

- 멀티 에이전트 라우팅: [Multi-Agent Routing](/concepts/multi-agent)
- 에이전트 워크스페이스: [Agent workspace](/concepts/agent-workspace)

## 예시

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## 아이덴티티 파일

각 에이전트 워크스페이스는 워크스페이스 루트에 `IDENTITY.md`를 포함할 수 있습니다:

- 예시 경로: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity`는 워크스페이스 루트(또는 명시적 `--identity-file`)에서 읽습니다

아바타 경로는 워크스페이스 루트를 기준으로 확인됩니다.

## 아이덴티티 설정

`set-identity`는 `agents.list[].identity`에 필드를 기록합니다:

- `name`
- `theme`
- `emoji`
- `avatar` (워크스페이스 상대 경로, http(s) URL, 또는 data URI)

`IDENTITY.md`에서 로드:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

필드를 명시적으로 재정의:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

설정 샘플:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```
